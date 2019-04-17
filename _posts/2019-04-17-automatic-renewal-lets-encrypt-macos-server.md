---
title: Automatic Renewal of Let’s Encrypt Certificates on macOS Server
---

I have a Mac running macOS Server providing several services behind a domain name that I'd like to secure with a certificate from Let's Encrypt, and have it automatically renew. Easy, right?

Not so much. Apple's many modifications to its implementation of Apache initially made integrating with Let's Encrypt difficult, and while most of the issues have been largely worked out, getting renewal to work reliably has been a sticking point. Although the numerous posts throughout the internet were helpful in writing this post, none offered a clean, up-to-date solution. It's a typical situation: some new technology has issues in specific situations, several posts appear with their own workarounds, fixes get implemented upstream, but the original posts are never updated. So here's my take on the subject.

## Ingredients

You will need:

- a Mac running some version of macOS Server. (Mine is still running OS X 10.11; yours should probably be something newer.)
- [Homebrew](https://brew.sh) installed.
- port 80 and 443 configured on your router to forward to your Mac.
- an internet connection with a fixed IP address.
- a domain name configured in public DNS to point to your IP address.

Use Homebrew to install _certbot_, which handles the creation and renewal of certificates from Let's Encrypt. (Older tutorials may refer to the package's original name, _letsencrypt_.)

    brew install certbot

Stop the built-in Apache webserver temporarily so that _certbot_ can bind to port 80 later. (Simply disabling the Websites service doesn't do that, nor does unloading _/System/Library/LaunchDaemons/org.apache.httpd.plist_, which isn't used by Server.)

    sudo launchctl unload /Applications/Server.app/Contents/ServerRoot/System/Library/LaunchDaemons/com.apple.serviceproxy.plist

## Creating the certificate

The following command will request a new certificate for _server.internal.company.ca_:

    sudo certbot certonly --standalone -d server.internal.company.ca

- _sudo_ allows the command to create its default log directory.
- _certonly_ obtains but does not install the certificate.
- _\-\-standalone_ runs a separate web server for the request process rather than relying on Apache.
- You can request a certificate for multiple domains by appending additional _-d_ arguments to the command.

Follow the prompts for an email address and agreeing to the terms of service. If all went well, you'll be told where to find your new certificate files, e.g. `/etc/letsencrypt/live/server.internal.company.ca/*`.

You may want to make its new log directory readable by users other than root. This command adds read and execute permissions for group and other, matching the permissions of other directories in `/var/log`.

    sudo chmod go+rx /var/log/letsencrypt

Finally, reload the web server we disabled earlier.

    sudo launchctl load /Applications/Server.app/Contents/ServerRoot/System/Library/LaunchDaemons/com.apple.serviceproxy.plist

## Automating the renewal

For the certificate renewal process to work, you'll need to modify its configuration. Because Let's Encrypt uses HTTP to authenticate our server during the renewal process, it'll have to use the macOS web server instead of its own, since only one process can use any port at a time. This is done by editing the automatically-generated configuration file for the certificate we just created, located within `/etc/letsencrypt/renewal`.

As root, edit the file (yours will be named according to your certificate's domain):

    sudo nano /etc/letsencrypt/renewal/server.internal.company.ca.conf

and find this line:

    authenticator = standalone

Change it to `authenticator = webroot` and add these lines to the end. Use your own domain and adjust the path if necessary, then save and close the file.

    [[webroot_map]]
    server.internal.company.ca = /Library/Server/Web/Data/Sites/Default

Certificates from Let's Encrypt expire after three months, and while you could just run `sudo certbot renew` manually, you're better off having it run for you at regular intervals using a system-wide launchd item.

Start by creating a new file:

    sudo nano /Library/LaunchDaemons/local.certbot.renew.plist

Insert the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Label</key>
        <string>local.certbot.renew</string>
        <key>RunAtLoad</key>
        <true/>
        <key>StartCalendarInterval</key>
        <dict>
            <key>Weekday</key>
            <integer>1</integer>
            <key>Hour</key>
            <integer>3</integer>
            <key>Minute</key>
            <integer>1</integer>
        </dict>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/local/bin/certbot</string>
            <string>renew</string>
            <string>--quiet</string>
        </array>
    </dict>
</plist>
```

Save and close the file, then load the new item into launchd:

    sudo launchctl load -w /Library/LaunchDaemons/local.certbot.renew.plist

Your Mac will now run `/usr/local/bin/certbot renew --quiet` each Monday at 3:01.

### Checking the output

If you want to see the command's output when run by launchd, add these lines to `local.certbot.renew.plist` just before the final `</dict>`.

```xml
        <key>StandardErrorPath</key>
        <string>/tmp/local.certbot.renew.log</string>
        <key>StandardOutPath</key>
        <string>/tmp/local.certbot.renew.log</string>
```

Then unload and load the item, whose `RunAtLoad` key being set to `true` makes it run the command as soon as it's loaded.

    sudo launchctl unload /Library/LaunchDaemons/local.certbot.renew.plist
    sudo launchctl load -w /Library/LaunchDaemons/local.certbot.renew.plist
    tail -f /tmp/local.certbot.renew.log

Note that _certbot_ also logs everything to `/var/log/letsencrypt/letsencrypt.log`, though.

## Importing the certificate

For Server to make use of the certificate, it needs to be imported into the macOS keychain. Here you'll create a script that'll be run by _certbot_ which converts the certificate to a keychain-friendly format before performing the import. By locating it in the `renewal-hooks/deploy` directory, it'll only be run on each successful renewal.

Create the script:

    sudo nano /etc/letsencrypt/renewal-hooks/deploy/keychain-import.sh

Insert the following code:

```sh
#!/bin/sh

[ $EUID -ne 0 ] && echo "Must be run as root." && exit 1

# Ensure nothing happens if /etc/letsencrypt/live is empty
shopt -s nullglob

for PEM_FOLDER in /etc/letsencrypt/live/*
do
    DOMAIN=$(basename $PEM_FOLDER)

    # Generate a passphrase
    PASS=$(openssl rand -base64 45 | tr -d /=+ | cut -c -30)

    # Transform the pem files into a p12 file
    openssl pkcs12 -export -inkey "${PEM_FOLDER}/privkey.pem" -in "${PEM_FOLDER}/cert.pem" -certfile "${PEM_FOLDER}/fullchain.pem" -out "${PEM_FOLDER}/letsencrypt_sslcert.p12" -passout pass:$PASS

    # Delete the current certificate from the keychain
    security delete-certificate -Z $(security find-identity -v -p ssl-server -s ${DOMAIN} | grep "1)" | cut -d " " -f 4) -t /Library/Keychains/System.keychain

    # Import the p12 file into the keychain
    security import "${PEM_FOLDER}/letsencrypt_sslcert.p12" -f pkcs12 -k /Library/Keychains/System.keychain -P $PASS -T /Applications/Server.app/Contents/ServerRoot/System/Library/CoreServices/ServerManagerDaemon.bundle/Contents/MacOS/servermgrd
done
```

Make it executable and run it:

    sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/keychain-import.sh
    sudo /etc/letsencrypt/renewal-hooks/deploy/keychain-import.sh

The output "1 identity imported. 2 certificates imported." indicates the script ran successfully.

If you now open Server.app and click on "Certificates", you should see your new certificate listed there. Assuming an SSL variant of your site already exists (check the list in "Websites"), select the new certificate from the "Secure services using:" popup to start using it.

## Items of note

- The launchd item uses a full path to _certbot_ because its parent directory, `/usr/local/bin`, is not in the `$PATH` of the environment used by launchd items. (It's possible to modify its environment with the `EnvironmentVariables` key, but this is more concise.)
- Each invocation of _certbot_ creates a new log file in `/var/log/letsencrypt`. If you want to run the script more often, you may want to clean out older log files regularly.
- I haven't tested this with multi-domain or multiple certificates, but I've no reason to believe it wouldn't work as long as appropriate renewal conf file modifications are made.
- According to its man page, each change to the keychain runs `certupdate` which in turn runs several other helper tools. For this reason it isn't necessary for us to stop and start the web server when a certificate is updated.
- I ran into [this bug](https://community.letsencrypt.org/t/mac-osx-server-import-le-certificate/5447/31) in OS X 10.11/Server 5.2. The background process responsible for updating `/Library/Server/Web/Config/Proxy/apache_serviceproxy_customsites.conf` whenever a certificate is imported into the keychain adds duplicate entries for each site other than the default defined in the Websites panel. If any of those sites has an SSL variant, entries are created for both the proper and the self-signed certificate, which causes Apache to simply stop working. The only workaround is to manually switch each site's certificate to the self-signed and then back using Server.app upon each renewal. macOS 10.12/Server 5.3 does not appear to have this issue.

## Sources

- <https://medium.com/@JoshuaAJung/managing-your-mobile-devices-in-the-cloud-using-apples-own-mdm-solution-8a588d9724b6> - my main source, although it covers much more than just certificate installation.
- <https://community.letsencrypt.org/t/complete-guide-to-install-ssl-certificate-on-your-os-x-server-hosted-website/15005?source_topic_id=33254&source_topic_id=40061>
- <https://mac.lytics.eu/install-ssl-certificate-on-your-mac-server/>
- <https://blog.barijaona.com/web/settingup_letsencrypt.html>
- <https://www.macstrategy.com/article.php?211> - see also this article's list of references.
