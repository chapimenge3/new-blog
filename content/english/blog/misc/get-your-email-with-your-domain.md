---
title: "Get your any-name@your-domain.com Email Address with Free Email Service"
meta_title: "Get your any-name@your-domain.com Email Address with Free Email Service using Cloudflare, Resend and Gmail Service"
date: 2024-05-05 10:23:39
image: "/images/blogs/misc/domain-based-email.jpg"
categories: ["Misc"]
author: "Chapi Menge"
tags: ["misc"]
draft: false
---

Hey My people! Do you have your own domain name and want to have an email address like __hey@your-domain.com__ to receive and send email with it? In this post, I'll show you how to get a free email address with your domain name using Cloudflare, Resend and Gmail Service.

{{< toc >}}

## Get your Domain if you don't have one

I am sure most developers have their own domain name but if you don't have one, you can get one from [Cloudflare](https://www.cloudflare.com/) or [Namecheap](https://www.namecheap.com/) or any other domain registrar. 

I have my domain name inside Cloudflare with makes it easy to manage my domain and cloudflare generously free developer services like Worker, Pages, D1, KV and many more you can check out [here](https://developers.cloudflare.com/).

## How does it work?

After you got your domain name, you can manage email addresses with DNS records mostly known as MX records. MX records are used to route emails to mail servers.[You don't need to know how it works, But incase you want to know more, you can check out this [link](https://www.cloudflare.com/learning/dns/dns-records/dns-mx-record/).

So once we are in control of our domain DNS records, the first thing we want is to receive emails under our domain name. We can use cloudflare service called [Email Routing](https://developers.cloudflare.com/email-routing/) and forward to your personal email address.

This means whenever somebody send you an email to __any-email@your-domain.com__, it will be forwarded to your personal email address like __your-name@gmail.com__. I know you might ask me how about sending email with my domain name? Don't worry, I will show you how to do that with Gmail Service and Resend Service.

## Set up Email Routing (Receive Email with your Domain Name)

1. Go to your Cloudflare dashboard and search for __Email Routing__ using __Go to__ search bar. For some reason, you might not see it in the dashboard, but you can access it directly adding __https://dash.cloudflare.com/${account-id}/${your-domain}/email/routing/overview__

2. Click Add __Destination addresses__ and add your personal email address and verify it by clicking the link sent to your email address.

{{< image src="images/blogs/misc/cf-destination.jpeg" caption="Cloudflare Destination" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Cloudflare Destination"  webp="false" >}}

3. Click `Routing Rules`, then there is two option here __Catch-all__ and __Specific Address__. Catch-all means all emails sent to your domain will be forwarded to your personal email address. Specific Address means you can specify which email address to forward to your personal email address.
    - __Catch-all__: people can send to __*@yor-domain.com__ and it will be forwarded to your personal email address. This is [MY RECOMMENDATION] since you can just give an email address related to the topic you are discussing. For example if a person from github want to contact you, you can give them __github@your-domain.com__ and it will be forwarded to your personal email address.
    - __Specific Address__: You can specify which email address to forward to your personal email address. For example, you can specify __hey@your-domain.com__ to be forwarded to your personal email address. 

    - Depends on your choice, you can choose one of them or both.
    
{{< image src="images/blogs/misc/cf-routing.jpeg" caption="Cloudflare Routing" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Cloudflare Routing"  webp="false" >}}

4. Click __Save__ and you are done with Email Routing. 

Now Half of the work is done, you can now receive emails under your domain name and you can ignore the rest and reply from your personal email address. But if you want to send email with your domain name, you can follow the next steps.

At this point, you can test by sending an email to your custom email you added above if you add catch all just use __hey@your-domain.com__ and you will receive the email in your personal email address.

{{< image src="images/blogs/misc/test-email.png" caption="Test Email" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Test Email"  webp="false" >}}

## Send Email with your Domain Name

To send email with your domain name, we need to use SMTP service. SMTP stands for Simple Mail Transfer Protocol. It's a set of rules that allow you to send emails. Gmail is amazing enough to let you add more email address to send email from it. So without further ado, let's get started. 

1. Signup for [Resend](https://resend.com/) and verify your email address. Resend is not free all the time, but they offer free plan for 3000 emails per month and 100 emails per day limit. Am sure you are not important person to send more than 100 emails per day 😂.

2. Once you signup Go to __Domain__ Section and click __Add Domain__. Add your domain name. Then it will ask you to add DNS records to your domain. Go to your Cloudflare dashboard and add the DNS records or your domain registrar dashboard, then manage DNS records. 

{{< image src="images/blogs/misc/resend_dns_verify.jpeg" caption="Resend Dashboard" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Resend Dashboard"  webp="false" >}}

3. Once you add the DNS records, click __Verify__ and you are done with Resend setup and you will see the status __Verified__ like below.

{{< image src="images/blogs/misc/resend_dns_verify.jpeg" caption="Resend Domain Verification" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Resend Domain Verification"  webp="false" >}}

4. Now we need to Generate an API key in Resend and copy SMPT details. Go to __Settings__ and click __API Keys__ and click __Generate API Key__. Copy the API key it won't be possible to see it again so make sure to save it in safe place.

{{< image src="images/blogs/misc/resend-api.jpeg" caption="Resend API Key" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Resend API Key"  webp="false" >}}

Once you generate the API Key, Now you can go to __Settings__ > __SMTP__ and copy the SMTP details from there but it is common so i will put the details here.

```bash
host: smtp.resend.io
port: 587
username: resend
password: your-api-key
```
Now go to Gmail and click __Settings Icon__ Next to your profile and click __See all settings__.

{{< image src="images/blogs/misc/gmail-settings.jpeg" caption="Gmail Add Email" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Gmail Add Email"  webp="false" >}}

Then Go to __Accounts and Imports__ Tab and click __Add another email address__ under the __Send main as:__ section.


{{< image src="images/blogs/misc/gmail-add-email.jpeg" caption="Gmail Add Another Email" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Gmail Add Another Email"  webp="false" >}}

Now one thing to notice here is we can't make the email address to be any name like we did in the Email Routing. So you can add any email address you want to send email from. For example, you can add __your-name@your-domain.com__ or whatever email that you want to reply with. 

{{< notice "info" >}}
Turn off the option **__Treat as an alias__**.
{{< /notice >}}

Enter the any email address you would like to send from, You can name any name as you like. 

{{< image src="images/blogs/misc/gmail-add-email-cred-1.png" caption="Gmail Add Another Email" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Gmail Add Another Email"  webp="false" >}}

Now put the SMTP details we copied from Resend and click __Add Account__.

{{< notice "info" >}}
Make sure to copy and paste the token you generated in the API Key in __password__ field.
{{< /notice >}}

{{< image src="images/blogs/misc/gmail-add-email-cred-2.png" caption="Gmail Add Another Email" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Gmail Add Another Email"  webp="false" >}}



Now let's test it and check if the email send from different email address and also you can check form Resend dashboard.

{{< image src="images/blogs/misc/email-test-from-my-domain.png" caption="Test Email" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Gmail Send Email"  webp="false" >}}


Go to Resend > Logs and check the logs and see the details.

{{< image src="images/blogs/misc/resend-log.jpeg" caption="Resend Log" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Resend Log"  webp="false" >}}

That's it! You have successfully set up your domain email address to receive and send email with it. If you have any questions, feel free to ask in the comments.