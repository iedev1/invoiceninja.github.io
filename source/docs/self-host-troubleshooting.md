---
extends: _layouts.docs
section: content
---

# Troubleshooting

## PDFs don't appear to be updating

If you are using Cloudflare, then most likely Cloudflare could be caching your static data. To force cache busting, edit your nginx.conf file and add in the following snippet

```bash
location ~* \.pdf$ {
    add_header Cache-Control no-store;
}
```

On Apache based servers, open the [/public/.htaccess](https://github.com/invoiceninja/invoiceninja/blob/master/public/.htaccess#L25) file and update the mod_headers block

```apacheconf
<IfModule mod_headers.c>
    # Blocks Search Engine Indexing
    Header set X-Robots-Tag "noindex, nofollow"

    # Prevents PDF File Caching
    <FilesMatch ".pdf$">
        Header set Cache-Control no-store
    </FilesMatch>
</IfModule>
```

## Email not sending

If you are experiencing issues sending emails be sure to double check your .env file contains the correct fields configured. 

```bash
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME="your_email_address@gmail.com"
MAIL_PASSWORD="your_password_dont_forget_the_quotes!"
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="your_email_address@gmail.com"
MAIL_FROM_NAME="Full Name With Double Quotes"
```

<x-warning>If you are using Gmail - Use an [app specific password](https://support.google.com/accounts/answer/185833?hl=en) or ensure you have less secure apps turned on.</x-warning>

<x-warning>If you are using Office 365 - You may need to [enable SMTP AUTH](https://docs.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission).</x-warning>

If you are using gmail smtp relay, then a additional .env variable is required.

```
SERVER_NAME=
```

Emails will fail with the message:

```
Expected response code 250 but got an empty response
```

without this additional field.

The ```MAIL_MAILER``` field defines which email driver you wish to use, this could be postmark, maildriver, smtp - anything that Laravel 8 supports natively is supported in this app.

If the mail config is correct, the next place to check would be to check the error logs for any errors that are being thrown, the error log is found in ```storage/logs/laravel.log```

The final source of information in diagnosing mail troubles is to inspect the ```System Logs``` tab in the dashboard of the application, in here we log any messages from the mail server itself which may be instructive as to the cause of your issues.

If you are using the Queue system ie. QUEUE_CONNECTION=database then you may also want to check the ```jobs``` table in the database, there should be _no_ records in that table... If there are records in the table it means that your queue is not running and therefore no mail jobs are being processed.

It's possible the emails are sent but are blocked for DNS, SPF, DKIM or other reasons. In these cases emailing a test invoice to [mail-tester.com](https://mail-tester.com) can help debug certain problems.

Also, if you see in /storage/logs/invoiceninja.log this line ```error failed with stream_socket_enable_crypto(): SSL operation failed with code 1. OpenSSL Error messages:
error:14090086:SSL routines:ssl3_get_server_certificate:certificate verify failed``` then try running `yum update` on your webserver, it should fix the ca-certificates problem.


## GoDaddy email sending woes

GoDaddy does not allowing sending via third party [SMTP](https://my.godaddy.com/help/send-form-mail-using-an-smtp-relay-server-953) servers. They require sending all email via their own servers. If you need to use GoDaddy, we suggest using a transactional email service such as PostMark to bypass.

## PDF conversion issues.

We strongly recommend using the built in [snappdf](https://github.com/beganovich/snappdf) package which is a highly performant PDF generator based on the headless chrome/chromium binary. This package is perfect for users that have root access to their server and are able to install the required dependencies if needed.

To configure SnapPDF use the following .env vars

```bash
PDF_GENERATOR=snappdf
```

Snappdf is also the default PDF engine in our [Docker](https://github.com/invoiceninja/dockerfiles) image, so if you prefer a very simple installation please consider our Docker setup as it is very fast to get going!

If you are on shared hosting, snappdf probably will be impossible for you to use as you do not have access to the subsystem to install the required packages. Instead, you will need to use a hosted PDF service, the two that Invoice Ninja v5 supports is [PhantomJS Cloud](https://phantomjscloud.com/) and our own hosted PDF generator that users with a white label license can use for _free_ to generate _unlimited_ PDFs.

### Phantom JS

Phantom JS Cloud is the default PDF engine [PhantomJS Cloud](https://phantomjscloud.com/) to generate your PDFs, the default API key that comes with a clean installation will no reliably generate PDFs, to ensure you can generate PDFs you should register for an API key on the phantomjscloud website and use this key in the .env file.

Phantom JS can be toggled on and off by setting the PHANTOMJS_PDF_GENERATOR to either TRUE or FALSE. The following .env variables are available for configuring PhantomJS.

```bash
PDF_GENERATOR=phantom
PHANTOMJS_KEY='a-demo-key-with-low-quota-per-ip-address'
PHANTOMJS_SECRET='your-secret-here'
```

The `PHANTOMJS_SECRET` can be any random value, it's used to bypass the client portal password.

<p>Once this has been done you'll need to refresh the config cache:</p>

```bash
php artisan optimize
```

If you experience errors with PDF generation, such as `500 Server error` or `Failed to load PDF document` or a continuous loading bar, you must get a PhantomJS key [here](https://dashboard.phantomjscloud.com/dash.html#/signup), Replace it with the prefilled key `a-demo-key-with-low-quota-per-ip-address` and run `php artisan optimize` again. (On shared hosting to clear the cache you will need to either empty to contents of /bootstrap/cache/* OR run https://your.website.com/update?secret=secret)

<x-warning>
For PhantomJS to work, your Invoice Ninja installation web address must be public; localhost installations or those on private networks won't be able to use PhantomJS Cloud.
</x-warning>

### Hosted Invoice Ninja PDF generation

If you are a white label user, then to enable the Invoice Ninja hosted PDF generator you will need to add a variable to the .env file as follows

```
PDF_GENERATOR=hosted_ninja
```  

The hosted ninja PDF generator is an offsite PDF generator hosted by Invoice Ninja, which operate similar to PhantomJS.

<x-warning>
Don't forget to refresh your cache (not needed for shared hosting!) with php artisan optimize.
</x-warning>

## Cron not running / Queue not running

<x-warning>
It can take up to an hour for the red warning triangle to disappear after correctly configuring your Cron.  

After making any changes to your cron setup you'll want to force a recheck of the cron setting. To do this navigate to http://url/update?secret=
</x-warning>

If you are faced with your recurring invoices not firing, or your reminders not sending, then most likely your cron job isn't working. The first thing is to make sure you have your cron jobs configured correctly by following the guide [here](https://invoiceninja.github.io/docs/self-host-installation/#cron-configuration-1) 

If you are using shared hosting, then will need to add an additional parameter to the cron command which looks like this:

```
cd /path/to/root/folder && /usr/bin/php -d register_argc_argv=On artisan schedule:run >> /dev/null 2>&1
```

This will force a recheck and if the cron is working the red error triangle will disappear.

## Platform specific issues

### General advice

When facing errors, first set `APP_DEBUG=true` in `.env` and execute `php artisan optimize` to get more extensive debug information.

### Erroneous data format for unserializing 'Symfony\Component\Routing\CompiledRoute'

<p>The most common cause of this issue is running multiple version of PHP, if the caches are built with a different version of PHP you may see the above error as differing versions of PHP may not be interoperable on the same installation. Ensure you are running the same CLI and Web PHP version to prevent any errors/</p>

### Unable to connect to database after installation

<p>You may need to restart the queue like this</p>

```bash
php artisan queue:restart
```

### Nginx: 413 – Request Entity Too Large

This error indicated that the client_max_body_size parameter in NGINX is too small, you will need to edit your nginx config and increase the size

```bash
client_max_body_size 100M;
```

### Proxy configuration.

For users that rely on configuring a reverse proxy, please consider this post on our forum which details steps which may assist in configuring a reverse proxy.

<a href="https://forum.invoiceninja.com/t/selfhosting-setup-failing/5651/8">Reverse Proxy Invoice Ninja</a>

### Problems with migration

If you are experiencing issues with the migration not running as expected please run through the following checklist:

 * Ensure directories are read/writable by the webuser (ie www-data)
 * Ensure the cron scheduler is running (and working) - You can verify it is working by inspecting the ```jobs``` table in the database, it should be empty
 * Inspect the log file /storage/logs/laravel.log for further information.
 * If you are still experiencing issues, turn on advanced logging by adding the following variable to your .env file. ```EXPANDED_LOGGING=true``` then optimize with ```php artisan optimize``` . Then attempt the migration again and afterwards inspect the log file in storage/logs/invoiceninja.log

### libatk.so not loading for Google

Pdf generation will not working using the inbuilt PDF engine without some subsystem dependencies, please consult this resource for the list of necessary libraries for each supported platform <a href="https://github.com/beganovich/snappdf#headless-chrome-doesnt-launch-on-unix">Snappdf required libraries</a>

### WebCron configuration

Some systems do not allow cron configurations, one work around is to use a web cron service which can hit a defined endpoint which executes the scheduler via a GET HTTP request. Invoice Ninja has implemented a small service to allow a webcron service to hit the end point:

```
https://domain.com/webcron?secret=
```

To configure the service, you need to add a .env variable

```
WEBCRON_SECRET=password
```

Define your own secret password and then re optimize the cache The service will then be activated.

### Installing in a subdirectory.

It is possible to install Invoice Ninja in a subdirectory outside the doc root, to enable this you will need to update the .htaccess file (only if you are using the Apache webserver),

```php
RewriteRule ^(.*)$ public/$1 [L]
```

should be updated to

```php
RewriteRule ^(.*)$ subdirectoryname/public/$1 [L]
```

### Endless setup loop

If you are finding that all your pre setup checks are passing however you keep falling back to the setup screen, this could indicate that you are missing the ```mysql-client``` library which is needed to perform the initial migration. If you are unable to install this for some reason (ie. XAMPP) then you'll need to run the migrations manually by entering the following at the command prompt

```
php artisan migrate:fresh --seed 
```

### flock() expects parameter 1 to be resource, bool given

This error is thrown from deep within PHP and indicates a permissions issue - most likely the public/storage and/or storage/ directory is not writable by the web user, depending on your platform, you'll need to run something like:

```
sudo chown -R www-data:www-data public/storage
```

and/or

```
sudo chown -R www-data:www-data storage/
```

### Unresponsive UI

If for some reason the UI becomes unresponsive, you may need to flush some subsystem configuration and rebuild. It is possible to do this by navigating to the `/update?secret=`  route, ie. https://invoiceninja.test/update?secret= This will perform a number of system clean ups and may resolve issues resulting from an incomplete upgrade. To protect this route, you are advised to add a .env pararameter `UPDATE_SECRET=a_secret_passcode` this will restrict the route to users with the UPDATE_SECRET passcode.

### Communication link failure: 1153 Got a packet bigger than 'max_allowed_packet'

If you are using the database for your queue's then sometimes you may see an error from MySQL

```
1153 Got a packet bigger than 'max_allowed_packet'
```

This indicates the insertion payload is bigger than MySQL is configured to handle! To work around this, you will need to increate the mysql variable

```
max_allowed_packet
```

To a larger value. Sometimes a value of 1024M is required.

It may also be wise to increase the variable

```
max_connections
```

as similar errors can be reported from the DB.

### 500 error when editing PDF templates

There was a [report](https://forum.invoiceninja.com/t/500-error-when-editing-pdf-invoice-templates-potential-fix/7067) 
from the user who solved 500 error on their server by disabling ModSecurity.

### 500 error when trying to login or edit company details

Try these steps to fix the 500 server error when trying to login or editing company details

1. Download the latest update from the [github releases](https://github.com/invoiceninja/invoiceninja/releases) (not `invoiceninja.zip` but `Source code (zip)`)
2. Upload the zip, extract the files and override them in your /public_html/ (Be careful to not override the .env file or all will be gone)
3. Login to your root and make sure first of all that all files are owned recursively by the user, ex. `sudo chown -R www-data:www-data dir/`
4. Run this command `cd /home/domain.com/public_html/invoiceninja/ && php artisan migrate` or simply `php artisan migrate` whatever works for you, select "YES"
5. If an error occurs like this one

```
PHP Fatal error:  Cannot declare class UpdateDesigns, because the name is already in use in /home/domain.com/public_html/invoiceninja/database/migrations/2021_09_16_115919_update_designs.php on line 0
In 2021_09_16_115919_update_designs.php line n/a: Cannot declare class UpdateDesigns, because the name is already in use
```

Delete that file and retry the command until it works and runs properly.

7. Once succeeded with step 5, run this command `cd /home/domain.com/public_html/invoiceninja/ && php artisan optimize` or simply `php artisan optimize` whatever works for you
8. Go to https://domain.com/update?secret=x to be sure the update worked, it should load the login screen and work, you should also be able to edit the company details again.

### Unresolvable dependency resolving [Parameter #0 [ array $options ]] in class App\Utils\CssInlinerPlugin

When changes are made to the container this can causes the cache to become stale in the application preventing it from booting. 

The solution is to clear the contents of the folder ```bootstrap/cache```, by either manually deleting files or by running ```/update?secret=``` which will also delete the contents of this directory. 
