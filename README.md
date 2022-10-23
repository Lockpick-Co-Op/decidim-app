# OSP App

Citizen Participation and Open Government application.

This is a base app for all OSP projects. It uses OSP's decidim version.

## Deploying the app

* heroku run rake db:migrate
* Set SEED=1 as ENV variable
* heroku run rake db:seed --app osp-decidim
* See (Setting up the application .3)


## Setting up the application

You will need to do some steps before having the app working properly once you've deployed it:

1. Open a Rails console in the server: `bundle exec rails console`
2. Create a System Admin user:
```ruby
user = Decidim::System::Admin.new(email: <email>, password: <password>, password_confirmation: <password>)
user.save!
```
3. Visit `<your app url>/system` and login with your system admin credentials
4. Create a new organization. Check the locales you want to use for that organization, and select a default locale.
5. Set the correct default host for the organization, otherwise the app will not work properly. Note that you need to include any subdomain you might be using.
6. Fill the rest of the form and submit it.

You're good to go!

### How to deploy

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)

1. Use the "Deploy to Heroku" button
1. Choose a name for the app, and organization and a tier
1. Fill in the required env vars.
1. Create the app
1. Enable Review Apps for this app (you'll need to create a Pipeline)


## Work Notes

Important Note: Organization admins can't do anything until they've accepted the admin terms, you'll get "you don't have permissions" errors on every action other than logging in until you click that.

### Domains

Organizations each need their own hostname, which means a domain or subdomain (maybe a subdirectory, but haven't tried that yet).

In Heroku, there is a setting for custom domains, which can be added to an application. Adding that, and setting a CNAME DNS record
so a particular domain routes to the Decidim app, is sufficient to see an org with that [sub]domain set as its hostname to be visible.

One example of this is the subodmain [phila-decidim-demo.davidginzberg.com](https://www.whatsmydns.net/#CNAME/phila-decidim-demo.davidginzberg.com) which is configured to point to a Decidim instance called [`decidim-poc-lpc`](https://phila-decidim-demo.davidginzberg.com) on Heroku. Changing your local hosts file to route to the appropriate application may also work.


### Rails Console Commands

To get the Rails console in heroku, go to the "more" menu in the top right of the app dashboard and run `bundle exec rails console`

This is particularly useful for setting up org admins. Once you have a working org admin (and have accepted the terms) you can manage
regular users within the application itself

Get the object representing organizations:

```Ruby
orgs = Decidim::Organization
organization.host = <for example, the URL of the Heroku instance without https://>
# To get only the first (by ID) org, use `Decidim::Organization.first`
```

Create a user (WIP):

```Ruby
# admin value defines whether user is admin of an org. System admins are created with Decidim::System as shown above
u = Decidim::User.new(admin: true)
# Produces the following object:
#<Decidim::User id: nil, email: "", created_at: nil, updated_at: nil, decidim_organization_id: nil, name: nil, locale: nil, avatar: nil, delete_reason: nil, deleted_at: nil, admin: false, managed: false, roles: [], email_on_notification: false, nickname: "", personal_url: nil, about: nil, accepted_tos_version: nil, officialized_at: nil, officialized_as: nil, newsletter_token: "", newsletter_notifications_at: nil, extended_data: {}, following_count: 0, followers_count: 0, notification_types: "all", admin_terms_accepted_at: nil, session_token: nil, direct_message_types: "all">

```

Modify the admin user of the first organization: (helpful if SMTP is not configured, to activate an org admin user)

```Ruby
u = Decidim::User.first
#set user properties, eg:... 
# ...password
u.password = <password string literal>
u.password_confirmation = <same password string literal>
# ...skip email-based confirmation
u.skip_confirmation!
u.tos_agreement = true

#when done, save the user object to the DB and you're good to go
u.save!
```


