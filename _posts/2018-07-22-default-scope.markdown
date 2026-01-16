ActiveRecord's default_scope can be a handy feature for the lazy programmer (A flag I proudly wave).  Rails developers can leverage it to add a default condition that will apply to all queries accessing a given model.  Let's say you're building a news site.  Every time you query for an article you only want to return published articles.  Instead of explicitly filtering by published articles with every query:

```ruby
  Article.where(published: true).find(1)
```

You can define a default_scope on the `Article` model to always filter by published articles:

```ruby
class Article < ActiveRecord::Base
  default_scope where(:published => true)
end
```
Now any query accessing Article will have the filter applied:

```ruby
  Article.find(1)
```

```sql
SELECT articles.* FROM articles WHERE articles.published = true
```

While I'm not a huge fan of default_scope, it is a useful ActiveRecord feature that you're likely to come across if you spend your days in Rails projects.

Myself and the rest of the App Platform team spent the last few months working on some major schema changes to the VTS database to maximize our ability to represent the many types of relationships that can exist between an owner and an asset.  As part of this work we updated the relationship between Users and Accounts.  Instead of a many-to-many relationship which allows a user to exist on multiple accounts, the new data model actually simplifies this relationship by ensuring every user is only on one account.  This meant getting rid of the idea of an `AccountUser` in the application by:

1. removing the account_users junction table in favor of a one-to-many relationship now described with a users.account_id foreign key

2. removing any references to account_users

A schema + data migration solves the first issue, and the second issue comes down to deleting code and who doesn't enjoy that?

As I said before default scopes can help the lazy programmer, but at VTS they have also served another purpose - security.  With some of the largest financial firms in the world over managing over 7 billion sqft on the platform you can bet [we take security seriously](https://www.vts.com/blog/uncategorized/vts-security-update-soc-2-eu-us-privacy-shield), both at the operations and application layers of the business.  We have a dedicated security team to handle things like our annual SOC2 audit, EU-U.S. Privacy Shield, etc, but at the end of the day security is everyone's responsibility.  Many in the industry describe security best practices as an onion, with many layers of protection stacked on top of each other.  The reality is that there's no such thing as a 100% secure application so when one layer inevitably fails you want to be sure you have another right behind it that has your back.  In our case this extra layer was implemented with default_scope.

We have come to really enjoy using [https://github.com/varvet/pundit] as our authorization system to define and check authorization of various resources.  This means that before a resource is served we pass it through Pundit to make sure the given user or destination has access to the object. For example before showing the currently signed in user a given Property, we have to make sure this user does indeed have access to this Property
