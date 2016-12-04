---
layout: post
title:  "Cassandra and Cloud 9"
date:   2016-12-04
categories: Cassandra
---
While attempting to get a Cassandra Socket IO program up and running, I came
across a strange issue.
The version of Cassandra that we were installing was using Cython. When doing

{% highlight ruby %}
sudo pip install cassandra-driver
{% endhighlight %}

errors were being generated relating to these Cython files. After scouring the 
internet I found a way around the issue using

{% highlight ruby %}
sudo pip install six futures
sudo pip install --install-option="--no-cython" cassandra-driver
{% endhighlight %}

This finally got my cassandra drivers to install properly for usage in python.
The only downside is that it wiped out my cassandra keyspaces from having to do
a clean install. 

After formating everything and putting it into an appropriate keyspace, my next
big hurdle was actually the data retrieval. Dr. Zacharski gave us some test code
to play with which was:

{% highlight ruby %}
session = cluster.connect('world')
rows = session.execute("select city, district, country from city where district = 'Arizona'")
for row in rows:
    print row.city, row.district, row.country
{% endhighlight %}

Now I looked at this similar to how python and postgres interact and came up with a 
base case of : 

{% highlight ruby %}
rows = session.execute("select city, district, population, countrycode from city where city = %s;", (searchTerm,))
        if (rows):
            for row in rows:
                print row.city, row.district, row.population, row.countrycode
                emit('citySearch', row)
{% endhighlight %}

Now the execution is similar to postgres so the queries went through just fine.
The next big issue that popped up was that the data wasn't being sent to socket IO
in a form that I could get socketIO to accept. Judging off of the print function
above, I was lead to believe that session.execute returned a dictionary, similar to 
postgres. After tinkering around I found that in order to get the data to pass as
a dictionary to socket, I had to assign it as such.

{% highlight ruby %}
result = {'city': row.city, 'district': row.district, 'population': row.population, 'countrycode': row.countrycode}
{% endhighlight %}

This was the final piece that brought my code together into working form. I am
still not entirely sure as to what the row variable is once session.execute runs.
It behaves like a dictionary, can be called like one, but does not pass as one.

I will look more into this in my free time to hopefully resolve the matter.

EDIT: After working with Cassandra I cannot in good conscience reccomend it to
anyone who needs reliability in their data sets. So far, just with 2 fairly simple
tables, I have had multiple crashes of Cassandra. I have also imported cql files
that were properly formatted (entered them in manually just fine) and there was
massive amounts of data loss (about 80% of the data didn't make it). From research
this isn't strictly a Cloud 9 issue, although attempting to run the resource heavy
Cassandra on a VM is frustratingly slow.

o7
-Roy