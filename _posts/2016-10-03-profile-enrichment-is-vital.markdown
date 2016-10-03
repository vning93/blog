---
layout: post
title: "Profile Enrichment Is Vital for Your Marketing and Sales"
description: Learn how to use Auth0's Social Login and upcoming Progressive Profiling Plugin to target customers more effectively
date: 2016-10-03 17:48
author: 
  name: Federico Molina
  url: https://twitter.com/federicomolina
  mail: federico.molina@auth0.com
  avatar: https://cdn.auth0.com/blog/profiles/fedemolina.png
design: 
  bg_color: "#2A5383"
  image: https://cdn.auth0.com/blog/profile-enrichment/logo.png
tags: 
- profile-enrichment
- social-login
- growth-hacking
related:
- 2016-07-08-why-your-project-estimations-are-always-wrong
- 2016-07-05-growth-hacking-is-dead-long-live-growth-hacking
- 2016-08-19-why-you-should-ab-test-everything
---

Good marketing hinges on communicating the usefulness of your product. That means knowing who your customers individually are and how they'll use your product. 

When customers first sign up on a site, the site recognizes them by their name and email. Maybe their IP address is registered and a few security questions, but that's it.

That's a pretty basic profile.

You can't tell your customers apart if they are all just a name attached to an email address. You have no idea what they need from your product, where they work, or who they are. If your company hosts videos for businesses, HR will use the product differently than developers. So why would you market to them the same way? 

To know your customers, and tailor your marketing messages effectively, you need to enrich your user profiles and start progressively profiling.

## Get to Know Your Customers Through Profile Enrichment 

According to MailChimp, segmented email campaigns see about a [15% higher open rate](http://mailchimp.com/resources/research/effects-of-list-segmentation-on-email-marketing-stats/) across the board when compared to non-segmented email campaigns. Tailoring your message to your different customers is key. The way you develop segmented lists is through profile enrichment.

Enriched data points, based on information from your customers, can be used for segmentation. They'll allow you to send better campaigns. Knowing more about your customers will help you determine how to market to them effectively and show how your product can do what they specifically need.

### How to turn an email address into a whole user profile

Auth0's [Social Login](https://auth0.com/learn/social-login/) gives you access to your users' personal information. When users log in through Facebook, Twitter, GitHub, or a variety of other platforms, you have access to everything from their hometown to their work history.

![Setting up Facebook as a Social Provider](https://cdn.auth0.com/blog/profile-enrichment/facebook-permisions.png)

Social Login can be used to gather the information necessary to create enriched customer profiles. [User metadata](https://auth0.com/docs/metadata?mkt_tok=eyJpIjoiTVRNME1tWTVZVFk1WmpjNCIsInQiOiJvSHA0V2gya0wzbFwvSUdnNW1VNlFLUkV2aGN5TlQ2NGFwSDR0TzZnSGZmMjNnMk1LSnRWNlV2cGx0YXdiZGJSejdpNHdQUDUxcHJ2WXBvazRFamF3TTJ0VFF2RE5TYXR1aitvN0Y0bE1hVkk9In0%3D), or the personal information you gathered from your users via their social media, can be stored on Auth0 in the [User Profile](https://auth0.com/docs/user-profile). You can then query against your user metadata within Auth0. 

Social media platforms also verify email addresses for users, meaning if you use Social Login you'll get a real email address. Not one of the fake ones many people use when they sign up for a web application.

![Blocking an user](https://cdn.auth0.com/blog/profile-enrichment/block-user.png)

Social login isn't the only way to create an enriched customer profile from an email address. [Clearbit](https://clearbit.com/), which conducts customer analysis and provides business-centered data APIs, can also give you personal information about your customers. They add 85 unique data points to a customer's profile, including where a person works, how many people are in their company, and their social media accounts. 

You can also use [Auth0 Rules](https://auth0.com/docs/rules) to query your customers through Clearbit's API and add the information to your user profile. Once you manually add Clearbit to your Auth0 account, you can automate your information storage process.

![Clearbit Profile](https://cdn.auth0.com/blog/profile-enrichment/clearbit-profile.png)

Now your customer is starting to look like a three-dimensional person with an enriched profile like the one above. Alex MacCaw, for example, is a company founder. If he is one of your customers, you can start asking yourself what a founder would use your product for and send him emails that tell him how to get the most use out of it. 

## Check-in With Your Customers Directly With Progressive Profiling 

Clearbit and Social Login are going to be ineffective in grabbing personal information if your users don't have a strong online presence or aren't on social media. [Progressive profiling](https://auth0.com/blog/progressive-profiling/), which asks customers questions about their personal information directly, works better in these cases. 

If profile enrichment is like doing deep research on all your customers, progressive profiling is like taking them out for coffee every so often. Instead of asking customers to fill out a massive form at the beginning of the lifecycle, progressive profiling staggers surveys over time. Since forms with too many cells can look daunting, causing users to bounce from the page, progressive profiling can both ease users in while also pulling information.

At first login, for example, you should ask users for:

* Their name
* Their email address, and
* A password.

At second login, or at purchase, you can push it a little farther and grab:

* Their address
* Their birthday, and 
* Their phone number

At third login, round it out and extract the last information you need:

* Their company,
* How many people work for them, and
* Their industry.

Progressive profiling grabs a little bit of information every time customers reach certain points in the product. This allows you to pose questions at key moments, like when users first sign up or after they complete a project, to grab specific data. 

Your marketing gets smarter over time as you are fed personal information from your customers. Soon, you can use Auth0's Progressive Profiling Plugin to grab this information.

## Auth0's New Plugin Will Put Progressive Profiling Right On Your Site

Auth0 is working on a plugin which will do your progressive profiling for you. It will allow you to [ask your users questions throughout](https://gist.github.com/mgonto/43fc8d2a119c2f9733942855926a9f48) your product's lifecycle. If, for instance, you want to know how many of your users are founders or other high-ranking company leaders, our plugin will let you ask them directly.

When you ask your customer a question, a small box will pop up in the corner of the screen asking users what their role is in their company. They can select from a list of options, like “junior employee” or “senior official.” Or, if their position isn't listed, they can manually type in their role. All of this information can be saved to your User Profile.

You can then use this information to send segmented emails and launch tailored marketing campaigns. That way you can speak to the different ways your customers might use your product based on what's required of them in their roles.

## Marketing With Enriched Profiles Will Help You Unlock Growth

Ultimately, having a fuller understanding of your customers will help your company grow and improve multiple parts of your business. Specifically, you'll be able to effectively segment your customers for custom marketing, increasing your sales potential. 

With the Auth0 plugin, you can start to see your customers individually. Enriched profiles at the touch of a button, in addition to strategic questions posed around key areas of your product, will further your understanding of your customers' motivations. The more you know your customers, the more compelling and actionable your marketing will be.
