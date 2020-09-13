# smart-resumes-with-jekyll
templates for  smart resumes with jekyll


# Building a Multipurpose Resume Hosted on GitHub Pages
Do you struggle with figuring out how best to summarize your diverse (and, perhaps, lengthy...) experience into a handful or two of machine-learning-friendly bullet points?
Are you applying to several different types of jobs, perhaps seeking to make a career change?
Are you an inveterate tinkerer who likes to play around with technology?
If so, consider using Jekyll Collections to make a "Smart" Multipurpose Resume.

Jekyll is a popular free command-line program for Windows, Linux, or Mac that generates static web pages from your plain text or markdown documents. Crucially, it is used by [GitHub's Pages platform](https://pages.github.com), which provides an option for free web hosting; or you can upload the pages to your own server.

# Jekyll Collections

Jekyll includes a neat concept called "Collections" of related documents.  In this example, I show how to use Jekyll Collections to create different views of your experience through whatever prism is most important at the moment.  [The core Jekyll documentation for Collections](https://jekyllrb.com/docs/collections/) is quite good and includes [a step-by-step tutorial](https://jekyllrb.com/docs/step-by-step/09-collections/).

To build a multipurpose "smart" resume, I will add two Collections to my local Jekyll website: Accomplishments and Employers. Then I will use Jekyll's Liquid template language to filter bullets in and out depending on situation.

The procedure is as follows.

Create directories _accomplishments and _employers in the Jekyll site root.  The underscores are required in the directory names.

Modify _config.yml by inserting the following anywhere:
```
accomplishments:
  output: true
  permalink: /:collection/:name

employers:
  output: true
  permalink: /:collection/:name

```
This makes Jekyll aware of the collections.

Each document in _accomplishments describes one of your noteworthy accomplishments. You should use the [now-standard X-Y-Z form](https://www.inc.com/bill-murphy-jr/google-recruiters-say-these-5-resume-tips-including-x-y-z-formula-will-improve-your-odds-of-getting-hired-at-google.html) of "Accomplished [X] as measured by [Y], by doing [Z]."

Here is an example of an accomplishment.  I called this document "software-review.md".
```
---
resume_goal: product_mgr
keywords: strategy, money
employer: Acme Widgets
---
Reduced company spending by approximately $700,000 (25%) and reduced business risk to my product by review of company-wide utilization of key software dependency that resulted in renegotiated contract with better terms.

```
Each document in _employers describes one of your past employers.  Here is the document for a prev employer:

```
---
layout: default
employer: Acme Widgets
from_date: 20170501
to_date: 20200318
---
Product Manager, Acme Widgets, May 2017 - March 2020
```
We know we will need to have a match field that enables us to match accomplishments with the employers that own them, so both "accomplishments" and "employers" have a field called "employer". Be sure to be consistent in spelling the values of employer in both collections!

We also know that we're likely going to want to sort employers chronologically, so we include simple from/to date fields.

You should be sparing in creating these "front matter" key/value pairs: just the bare minimum of metadata that you will need for template logic.  Unstructured content follows the second set of dashes.

# The Liquid templating language

Now we create the "smart" part of the "smart resume."  We need some logic to pick out only the accomplishments that are relevant to the goal of the resume.  Jekyll uses the Liquid templating language which can do simple conditional logic and control operations inside HTML pages. 

To create a resume that emphasizes accomplishments relative to our goal, we are going to loop over the collection of employers, then look at each accomplishment belonging to that employer and determine whether it is relevant to our goal.  We accomplish this with nested for loops.

We are going to start by creating a resume for our hypothetical product manager who is considering a potential career change to developer relations.  We will go through her accomplishments and pull out the ones that are relevant to this goal.  The value of "resume_goal" in the front matter for relevant accomplishments will be "developer_rx". The page will be called developer_resume_relations.html and stored in the Jekyll root.  It will have some Jekyll "front matter" protected by triple scores and including a goal statement at the beginning.

```
---
layout: default
title: Developer Relations Goal
---
Goal: a position as a developer relations advocate
```
Then the page will proceed through its logic.  

We'll use a Jekyll include at the top to provide our coordinates.

{% raw %}
`{% include coordinates.html %}`
{% endraw %}

Then we'll create a variable to allow us to re-use this template for other goals.

{% raw %}
`{% assign targetjob = "developer_rx" %}`
{% endraw %}

And we initialize a variable to keep track of whether a bullet is the first one per employer.

{% raw %}
`{% assign first_accomplishment = true %}`
{% endraw %}

We want our list of employers arranged in reverse chronological order, so we create a new array.

{% raw %}
`{% assign employersbydate = site.employers | sort: "to_date" | reverse %}`
{% endraw %}

Then the outermost for loop iterates over the list of employers.

{% raw %}
`{% for employer in employersbydate %}`
{% endraw %}

The next for loop iterates over the accomplishments.
{% raw %}
`{% for accomplishment in site.accomplishments %}`
{% endraw %}

And, since each accomplishment can pertain to multiple resume goals, we have to loop over them, too.
{% raw %}
`{% for goal in accomplishment.resume_goal %}`
{% endraw %}

Now we do a logic check to determine whether each accomplishment should be "published" in this version of the smart resume. If it is the first accomplishment belonging to the employer, we print the main body (.content) for both employer and accomplishment in bulletized form, then flip the "first accomplishment" variable to "false."

{% raw %}
```
{% if goal == targetjob and employer.title == accomplishment.employer and first_accomplishment == true %}

{{ employer.content }}

<ul>

    <li>{{ accomplishment.content }}</li>


    {% assign first_accomplishment = false %}
```{% endraw %}

If the logic check says that the accomplishment is related to the goal but it's not the first accomplishment for that employer, we just print the accomplishment.  And if the accomplishment is not related to the goal at all, we just skip to the next iteration of the for loop.

{% raw %}
```
{% elsif goal == targetjob and employer.title == accomplishment.employer and first_accomplishment == false %}

    <li>{{ accomplishment.content }}</li>

    {% assign first_accomplishment = false %}

    {% else %}

    {% endif %}
```{% endraw %}

Then we close each of the for loops in turn.
{% raw %}
```
    {% endfor %}

    {% endfor %}
</ul>

{% assign first_accomplishment = true %}

{% endfor %}
```{% endraw %}

And add the Education section.
{% raw %}
`{% include education.html %}`
{% endraw %}

Now let's put this [all together](https://gist.github.com/fredzannarbor/53e765296504ffaf8c7ca7a32ffd8467) and try a test run.  

Magically, we see a resume that contains a list of all [my accomplishments pertaining to developer relations](/resumes/developer_rx_resume.html), sorted in reverse chronological order by employer.

# Further Information

The html pages containing the liquid togic and a few sample collection documents in markdown are [available on GitHub.]( https://github.com/fredzannarbor/smart-resumes-with-jekyll)

