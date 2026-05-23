# Additional Setup Notes 

This document contains additional setup notes for the Moodle installation used in the **EU Railways Academy** prototype.

It focuses on three technical parts:

- Composer
- HTTPS setup with Certbot
- AI-Provider (Ollama)

These steps are separated from the main Moodle installation guide because they are not always part of the basic Moodle installation, but they were important for this project.

---

## 1. Composer Setup

Composer is a dependency manager for PHP.

In Moodle projects, Composer can become important when:

- Moodle dependencies need to be checked
- plugin dependencies are required
- PHP package compatibility needs to be inspected
- development or plugin-related commands are used

Composer was especially relevant in this project because some compatibility problems appeared between:

```text
Moodle version
PHP version
Composer dependencies
plugin requirements
operating system packages
````

This showed that the newest PHP version is not always the best choice for a Moodle setup.

---

## 2. Install Composer

Composer can be installed with:

```bash
sudo apt update
sudo apt install composer -y
```

Check the installed Composer version:

```bash
composer --version
```

Check the PHP version used by Composer:

```bash
php -v
```

This is important because Composer uses the installed PHP version when resolving dependencies.

---

## 3. Composer and PHP Version Compatibility

During this project, one important lesson was that Composer may fail if the installed PHP version is too new or not supported by some Moodle dependencies.

For example, dependency conflicts can happen with packages such as:

```text
ezyang/htmlpurifier
openspout/openspout
```

The important point is not that “newer PHP is bad”.

The better explanation is:

```text
The Moodle version, PHP version, Composer dependencies, and plugin versions must be compatible with each other.
```

Before changing PHP or Moodle versions, always check:

```bash
php -v
composer --version
```

Inside the Moodle directory, Composer-related files may include:

```text
composer.json
composer.lock
```

These files define or lock the PHP package dependencies.

---

## 4. When Composer Problems Happen

If Composer reports dependency conflicts, check the following:

```text
1. Which PHP version is installed?
2. Which Moodle version is installed?
3. Which plugin version or branch is being used?
4. Does the plugin officially support this Moodle version?
5. Does the PHP version match the supported Moodle requirements?
```

Useful commands:

```bash
php -v
```

```bash
composer diagnose
```

```bash
composer validate
```

If the error is caused by incompatible PHP or plugin dependencies, the better solution is usually not to force Composer.

Instead, choose a compatible combination of:

```text
Moodle version
PHP version
plugin branch
operating system packages
```

---

## 5. Important Composer Lesson from This Project

For the EU Railways Academy prototype, Composer showed clearly that version planning matters.

The setup should not start with:

```text
Install the newest Moodle and newest PHP first.
```

The better approach is:

```text
1. Check which plugins are required.
2. Check which Moodle versions the plugins support.
3. Choose the Moodle version.
4. Choose a compatible PHP version.
5. Then install Moodle and dependencies.
```

This avoids dependency conflicts later.

---

# HTTPS Setup with Certbot

HTTPS was used so that the Moodle website could be accessed securely.

For this project, HTTPS was configured using:

```text
Apache2
Certbot
Let's Encrypt certificate
```

HTTPS is important because Moodle includes login, user data, course data, and administrator access.

A Moodle site should not be used publicly without HTTPS.

---

## 6. Install Certbot

Install Certbot and the Apache plugin:

```bash
sudo apt update
sudo apt install certbot python3-certbot-apache -y
```

Check Certbot:

```bash
certbot --version
```

---

## 7. Prepare Apache Before Running Certbot

Before running Certbot, the Apache virtual host should already be working with HTTP.

Example Apache configuration:

```apache
<VirtualHost *:80>
    ServerName your-domain.com
    DocumentRoot /var/www/moodle

    <Directory /var/www/moodle>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/moodle_error.log
    CustomLog ${APACHE_LOG_DIR}/moodle_access.log combined
</VirtualHost>
```

Enable the site:

```bash
sudo a2ensite moodle.conf
```

Reload Apache:

```bash
sudo systemctl reload apache2
```

Check Apache status:

```bash
sudo systemctl status apache2
```

The domain should already point to the server before Certbot is executed.

---

## 8. Request HTTPS Certificate

Run Certbot with Apache support:

```bash
sudo certbot --apache
```

Certbot will ask for:

```text
email address
domain name
HTTP to HTTPS redirect preference
```

For this project, HTTPS redirection was useful because visitors should automatically use the secure version of the Moodle website.

The expected result is that Certbot creates or updates an Apache SSL configuration file.

Example:

```text
/etc/apache2/sites-enabled/moodle-le-ssl.conf
```

---

## 9. Check HTTPS Configuration

After Certbot is finished, check Apache:

```bash
sudo apache2ctl configtest
```

Expected result:

```text
Syntax OK
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

Open the website in the browser:

```text
https://your-domain.com
```

The browser should show a valid HTTPS connection.

---

## 10. Automatic Certificate Renewal

Let's Encrypt certificates expire regularly, so renewal must work.

Certbot usually installs automatic renewal through a systemd timer.

Check the timer:

```bash
systemctl status certbot.timer
```

Test renewal:

```bash
sudo certbot renew --dry-run
```

If the dry run is successful, automatic renewal should work.

---

## 11. Apache HTTPS File

After Certbot, Apache may contain a separate HTTPS configuration.

Example structure:

```apache
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName your-domain.com
    DocumentRoot /var/www/moodle

    <Directory /var/www/moodle>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    SSLCertificateFile /etc/letsencrypt/live/your-domain.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/your-domain.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>
```

Do not upload real certificate files or private keys to GitHub.

---

## 12. Moodle Configuration for HTTPS

After HTTPS is enabled, Moodle should use the HTTPS address in `config.php`.

Example:

```php
$CFG->wwwroot = 'https://your-domain.com';
```

If this is still set to `http://`, Moodle may redirect incorrectly or load some resources insecurely.

After changing `config.php`, purge Moodle caches:

```bash
sudo -u www-data php /var/www/moodle/admin/cli/purge_caches.php
```

The Moodle path may be different depending on the installation.

---

## 13. Security Reminder

For GitHub, it is acceptable to document the Apache and Certbot setup.

However, never upload:

```text
SSL private keys
real API keys
database passwords
Moodle config.php with real credentials
private SSH keys
full database dumps with real user data
```

Use example files instead:

```text
apache-moodle-example.conf
config.example.php
.env.example
```

---

## 14. Short Summary

Composer was important in this project because it revealed dependency and version compatibility issues between Moodle, PHP, plugins, and PHP packages.

Certbot was used to enable HTTPS for the Moodle website, which is necessary for a public Moodle platform.

The key lesson is:

```text
Moodle deployment is not only about installing Moodle.
It also requires careful version planning, dependency checking, HTTPS configuration, and secure handling of configuration files.
```

# AI Feature with Ollama

Besides the content translation feature, the Moodle prototype also tested Moodle’s AI functionality with a locally hosted Ollama server. Moodle introduced an AI subsystem from Moodle 4.5 onward, where administrators can configure AI providers for actions such as text generation, summarisation, and image generation. Moodle’s AI provider system is designed to support different providers, including OpenAI-compatible APIs and local/self-hosted models such as Ollama. The official Moodle 4.5 announcement also explains that Moodle’s AI subsystem is intended to give institutions more control over how AI is used in their learning platform.

**References**:

```text
AI in Moodle 4.5:
https://www.youtube.com/watch?v=pfpK7SJTPks

Moodle + Ollama: Adding your own self-hosted AI to your Moodle:
https://youtu.be/_9VfQ-5_Qyk

Moodle AI Providers documentation:
https://docs.moodle.org/en/AI_providers

Ollama OpenAI compatibility documentation:
https://docs.ollama.com/api/openai-compatibility
````

In the Moodle Site administration, there is an option to configure AI providers. In theory, Ollama can be used directly as an AI provider. However, in this Moodle version, the direct Ollama provider configuration did not work reliably in my setup. Because of that, I used the OpenAI provider instead and configured it to communicate with the local Ollama server through Ollama’s OpenAI-compatible API.

The configuration was done through:

```text
Site administration
→ General
→ AI
→ AI providers
```

or depending on the Moodle interface:

```text
Site administration
→ Plugins
→ AI
→ AI providers
```

The OpenAI provider was selected, but instead of connecting it to the real OpenAI API, the API endpoint was adjusted to point to the local Ollama server.

Example endpoint format:

```text
http://<ip-address>:11434/v1
```

If Ollama runs on the same machine as Moodle, the address may be:

```text
http://localhost:11434/v1
```

If Ollama runs on another machine in the local network, the local IP address of that machine has to be used instead:

```text
http://192.168.x.x:11434/v1
```

The important part is that the Ollama server must be reachable from the Moodle server. Ollama normally uses port `11434`, and the OpenAI-compatible API path uses `/v1`.

The API key field in Moodle also had to be filled in, even when using Ollama locally. The value itself was not important in this setup, but Moodle expected the field not to be empty. Therefore, a placeholder value could be used.

Example:

```text
API key: anything
```

After that, each AI action setting had to be adjusted. The model name had to match a model that was already installed in Ollama. For example:

```text
llama3.1
```

or another locally installed Ollama model.

The actions for text generation and summarisation could be enabled, but image generation had to be disabled. At the time of this setup, Ollama was not able to generate images in the same way required by Moodle’s image generation action. Therefore, enabling image generation would cause problems or would not produce the expected result.

Recommended action configuration:

```text
Generate text: enabled
Summarise text: enabled
Explain text: enabled
Generate image: disabled
```

It is also important to check the IP address and port carefully. If the Moodle server cannot reach the Ollama server, the AI feature will not work. When Moodle and Ollama run on different machines, firewall settings and network accessibility must also be checked.

### Security Note

The Ollama server should not be exposed openly to the internet without protection. If the local AI server is reachable from outside, there is a risk that other users may send unwanted or harmful prompts to it. This is especially important because AI systems can be vulnerable to prompt injection, where malicious instructions are hidden inside user input or content. Therefore, the Ollama endpoint should be kept local or protected with proper firewall rules, access control, HTTPS/reverse proxy configuration, and server hardening. In AWS there is a Firewall feature called WAF and it can filter the prompt injection.

For this prototype, Ollama was mainly tested as a self-hosted AI option to reduce dependency on external cloud-based AI providers. It showed that Moodle can be extended with local AI functionality, but it also made clear that AI integration requires careful configuration and security consideration.



