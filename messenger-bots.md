## Creating a messenger bot with express

Facebook announced recently the option to create [bots for their messenger platform](http://newsroom.fb.com/news/2016/04/messenger-platform-at-f8/).

> Every month, over 900 million people around the world communicate with friends, families and over 50 million
businesses on Messenger. It’s the second most popular app on iOS, and was the fastest growing app in the US in 2015.

There's an API to receive and send messages, this way you can engage with users and customers programatically,
there's a few steps to get it working, so we recommend starting with a simple application. You interact with the users
as a page, not as an individual, so during the process you will be creating a Facebook app and page.

There's a [sample application](https://github.com/dvidsilva/messenger-bot) with two endpoints, one to receive and
another to send messages. On this application the sender will receive the same message it send. We're using the
same url for both, when a message is received the request is GET and to send a new message it will be using
POST requests.

## Overview

1. Setup a Nodejs application.

2. Facebook does not play nice with self signed certificates. Use one from LetsEncrypt.

3. Follow steps mentioned at Facebook Messenger API docs to create an app and a page required.

  * Generate page access token.

  * Use token generated in previous step and subscribe app to page.

5. You are ready to go, bot should respond back with exact string sent.

6. Other steps

### Setup a Nodejs application

The easiest way to do this is creating a droplet in digital ocean, while creating the droplet you can
select a small instance, and nodejs on the one click applications list.

Running a nodejs production ready application should be the easiest part for you. Clone the
[sample application](https://github.com/dvidsilva/messenger-bot) and run it with [pm2](http://pm2.keymetrics.io/).

You can find more information in the [digital ocean guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-14-04).

The application needs to have two endpoints, one get and one post.

Get will be used by Facebook during the setup process to verify that you own this endpoint.

Post will be used every time a user sends you a message, with the payload of the message.

To send a message you will make a Post request to Facebook with the content.

The verify endpoint looks like this:
```
if (req.query['hub.verify_token'] === conf.VERIFY_TOKEN)
  return res.send(req.query['hub.challenge']);
```
It makes sure the token sent by Facebook matches the one in your configuration, you will be setting this up later.


### Create a certificate from LetsEncrypt

[https://letsencrypt.org/](LetsEncrypt) let's you create a certificate for ssl. It’s free, automated, and open.
This certificates will be good enough for your bot.

To prepare for this process, create an A record for the domain or subdomain that you want to use.

Clone the LetsEncrypt repo

```
sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
```

And generate a certificate

```
cd /opt/letsencrypt
./letsencrypt-auto certonly -a webroot --webroot-path=/usr/share/nginx/html -d your.domain.com
```

It will ask you to enter an email address to get information about your domain and the certificate, and
agree to the terms of service.

**Using the Webroot Plugin** The Webroot plugin works by placing a special file in the /.well-known directory 
within your document root, which can be opened (through your web server) by the Let's Encrypt service for validation.
Depending on your configuration, you may need to explicitly allow access to the /.well-known directory.

Modify your nginx configuration by adding this to the server block


```
sudo vi /etc/nginx/sites-available/default
```

```
listen 443 ssl;

server_name  your.domain.com;

ssl_certificate /etc/letsencrypt/live/your.domain.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/your.domain.com/privkey.pem;
```

You can find more detailed steps [in the digital ocean guide](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encryp
t-on-ubuntu-14-04).


### Follow steps mentioned at Facebook Messenger API docs

The full Facebook guide can be found [here](https://developers.facebook.com/docs/messenger-platform/implementation).

You need to have a page, your users will be messaging a page and not an individual, so create a new page.
Specially for your testing attempts, is better to use something not important and not risk an important business
page. In my case I created [Messenger Echo Bot](https://www.facebook.com/Messenger-Echo-Bot-1747759535458893)

On the app configuration, you will generate a token, add this token to [conf.js](https://github.com/dvidsilva/messenger-bot/blob/master/conf.js).

To generate the token, go to the product settings of the app configuration, chose to add a product and find Messenger, there it will let you select
a page you own and generate a token. Add this token as value in the configuration to `PROFILE_TOKEN`.

You can use this token to associate the page with the app, either on the app settings, or by running this command:

```
curl -ik  -X POST "https://graph.facebook.com/v2.6/me/subscribed_apps?access_token=<PROFILE_TOKEN>"
```

Create a random, safe, string, and add it to `VERIFY_TOKEN`. In the app configuration, you will be prompted to enter this
when adding a webhook. Your node app should be working by now, because Facebook will only let you add the webhook if this

When your app is in Development Mode, plugin and API functionality will only work for admins, developers and testers of the app. 
After your app is approved and public, it will work for the general public.

Remember to restart your app after editing conf.js, the fastest way to do it is `pm2 restart all`, tho be careful
if more apps are running on that server.

### Try it out

Go to the page you created and select the message option, the page will reply with the same text you sent.

If there are errors, debugging should be easy, there's just a handful of things that can go wrong.

Tail the logs for your app and look at what happens when you send a message to the page.

If nothing comes in the logs, it can be due to: the user not being associated to the app, thus not triggering the hooks,
or the hooks not being configured correctly.

The other common error is OAuthException, this means the `PROFILE_TOKEN` is incorrect.

If you run into other errors you can't fix, try posting a comment, opening a ticket or getting in touch and I'll try to
take a look.

### Other steps

To get your app ready for users you must provide a privacy policy, app icons, and a video of how your bot works.
Is quite a lot of work for test apps, and they seem to be taking this seriously and reviewing each of the submissions,
so make sure you go through all the steps before applying.

Your app would probably get rejected if this is all it's functionality, so try adding some clever jokes or more information.

You can find detailed instructions in the App Review part of your Facebook app settings.

Hope this helps to get you started, let us know what cool bots have you come up with! 

