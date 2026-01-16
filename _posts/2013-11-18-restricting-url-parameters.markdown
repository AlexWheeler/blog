I’m working on a job board for Boston area residents to post odd jobs for college students to complete.

When a user is filling out the form to post a new job they must decide which category their job falls under. For example if a user needs a student to help them with moving furniture they select the Moving category.  If they need a babysitter for the weekend they select the Child Care category.  Since category is an attribute of a job object I am able to query for jobs according to category.

![category](/assets/job-board-category.png)

I have a dynamic route at `/:category` that directs users to the index action of the jobs controller and sets `params[:category]` to `/:category`.

*routes.rb*

```ruby
match "/:category", to: "jobs#index", as: :category
```

The header tag for the index view is dynamic to be params[:category].  For example if a user navigates to */Moving* the index view is displayed with header Moving, listing all of the moving jobs.

![moving-index](/assets/job-board-moving-index.png)

This is great! A user can find jobs based on a certain category by visiting `/:category` and easily tell which jobs they are viewing by looking at the large, bold header or the url. Then I came across a problem.  Since the route and and header tags were dynamic the user could pass any value into the url and my job board had no way of determining if this was a valid category or not.  Users would be directed to a view with a header matching params[:category] whether it was a valid category or not.

Let’s say the user navigated to */Foo*. They would see the index view with a header Foo and a blank table.

![job-board-moving-foo](/assets/job-board-moving-foo.png)

I needed to come up with a way to validate that a category was valid or not.  My Job Board had to know that *moving* was a valid category and *foo* was not.

Luckily for me I had been playing around with a lot of methods that Ruby’s [Enumerable Module](https://ruby-doc.org/core-2.4.1/Enumerable.html) provides us with and remembered an awesome `include` method.  You can call `include` on any collection and pass it a parameter, and ruby checks if the collection contains this parameter.  It returns a boolean value, true if it does include the value, and false if it does not.

Perfect! All I need to do is make a collection of all of the valid categories, iterate through it and check to see if it includes `params[:category]`.  So, I did exactly that.

*jobs_controller.rb*

```ruby
def index
  categories - %w(child-care computer-help general home-services
									moving party-help-catering research-focus-group yardwork)
  if categories.include?(params[:category])
    @jobs = Job.where(category: params[:category])
  else
    render "jobs/not_a_category_error"
  end
end
```

I added an array to the index action of the jobs controller, checked to see if the category was valid, and used a conditional statement to dictate where to direct the user .  If the value is a valid category (returns true) it lists all of the jobs of the given category.  If the value is not a valid category (returns false) the user is redirected to an error page.

*not_a_category_error.html.erb*

![error](/assets/job-board-error.png)

Part II: Refactoring

Great, everything is working.  Now time for some refactoring.  Rails has many conventions or best practices, one of which is known as “skinny controller, fat model.”  Following this convention, I decided to extract the categories into the Job model and make it a constant.

*job.rb*

```ruby
class Job < ActiveRecord::Base
  CATEGORIES = %w(child-care computer-help general home-services
							 	  moving party-help-catering research-focus-group yardwork)
```

Now I am able to access this constant using Ruby’s :: operator and do the same sort of validation as before.

*jobs_controller.rb*

```ruby
def index
  if Job::CATEGORIES.include?(params[:category])
    @jobs = Job.where(category: params[:category])
  else
    render "jobs/not_a_category_error"
  end
end
```

Read Parameterizing object attributes, to learn further how to refactor this code.
