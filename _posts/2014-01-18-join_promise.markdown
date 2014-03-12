---
layout: post
title: Promises, noSql, joins... and more promises
---

###Intro


Two of the top trending terms of the last couple of years in the tech world are, sure enough, promises and nosql.
If you are into functional programming, the day will come that you get introduced to futures (deferred) and promises. Here is the thing: I'm not talking about __Scheme__ or __Scala__; if you ever had to deal with __JavaScript__, you are into functional programming - if you haven't realized that yet... [this](https://www.youtube.com/playlist?list=PL7664379246A246CB) is a good starting point.
On the other hand, you can not possibly not have at least heard of noSql databases and call yourself a developer. Of course, they are not the answer to every problem (and we will see an example of this tradeoff in a minute), but in many situations where scalability is of the essence, they often are the best choice.

Now, you might wonder, how are these two subjects possibly related? Well, first of all, often noSql datastore servers offers asynchronous calls that either natively behaves as promises or can be wrapped into deferred to mimic promises. [Redis](http://redis.io), for example, lets you pipeline your commands (which also gives a significant performance improvement), although it forces you to use a somewhat cumbersome and odd syntax.

However, as hard as it can be to believe, I want to talk about what I think it is another contact point between the two of them: joins.

####Promises

Let's take a step back, first, to briefly describe what promises and noSql DBs are. In particular, I'm going to refer to jQuery implementation of the [Promise/A](http://wiki.commonjs.org/wiki/Promises/A) proposal, and Google App Engine datastore.

Simply stated, a promise represents a value that is not yet known, and a future (deferred) represents work that is not yet finished.
A promise is basically a placeholder for a result which is initially unknown, and can be either be resolved or fail; they are a way to deal with asynchronous code flow, as for example _ajax calls_, animations, and so on; you could use callbacks to deal with async programming, but promises allows you a cleaner code style (which in turn means it's going to be easier to maintain your code) and provides a much easier way to combine different steps in the asynch workflow.
If you need to get in deeper into the subject, I found this two posts very clear:

* [Promise & Deferred objects in JavaScript Pt.1: Theory and Semantics](http://blog.mediumequalsmessage.com/promise-deferred-objects-in-javascript-pt1-theory-and-semantics)

* [Promise & Deferred Objects in JavaScript Pt.2: in Practice](http://blog.mediumequalsmessage.com/promise-deferred-objects-in-javascript-pt2-practical-use)

Also, if you have some spare time, some theory and the __Scala__ approach are very extensively explained in the [Coursera](http://coursera.org/) course about [Reactive programming in Scala](https://class.coursera.org/reactive-001).

####NoSql

While promises/futures have been around since the 70s (although only in the last few years they became widely used), NoSql databases are a more recent idea.
For decades, since relational databases where invented, we were thought that thr third normal form (or at least some kind of normal form) was the goal every DB architect should aspire to.
With the raise of Ajax, web applications has kind of taken over the market; distributed application have different kind of needs in term of availability and responsiveness (not to mention security), and as the average number of users they serve grew, scalability became increasingly important, so much that failing to scale would determine the thrive or death of a web site. Under these considerations, the assumptions on the DBs started to be rethought, questioning our absolute need for relation integrity. 
The main reason for that is performance: maintaining relations between different tables, turns out to be pretty costly when you need to split your DB to several distributed servers, in order to improve your response time, to allow for backups copy, and to overcome the space limitations of a single server (which inevitably you will hit if your company is hyper-growing).
It turns out that we do not always need a relational DB, nor we always need (or can) guarantee the __A.C.I.D.__ properties (we can soften some of them, depending on our needs).
To go back to data relations, very often web companies just need to store key-value pairs, or schema-free lists of entities totally unrelated among them. When this happens, noSql DBs represent a valid alternative: long story short, they can be easily split and duplicated, and so scaling them is much more easily; this means they can provide you with better performance, which in turn means your website will be responsive and users won't leave and you'll stay in business.
In practice, there is always a tradeoff among many aspects, and you'll have to carefully review each of them to decide the best solution for your business.

If you want to get further information about this subject, I suggest investing your time and money into two great books:

* [Scalability Rules: 50 Principles for Scaling Web Sites](http://www.amazon.com/Scalability-Rules-Principles-Scaling-Sites/dp/0321753887/ref=sr_1_1?ie=UTF8&qid=1389955981&sr=8-1&keywords=50+scalability+rules)

* [The Art of Scalability](http://www.amazon.com/The-Art-Scalability-Architecture-Organizations/dp/0137030428/ref=sr_1_1?ie=UTF8&qid=1389955938&sr=8-1&keywords=the+art+of+scalability)

Also, be sure to check this great talk: [Google I/O 2012 - SQL vs NoSQL: Battle of the Backends](http://www.youtube.com/watch?v=rRoy6I4gKWU&list=WLB663AFEDE79B3691)

####NoSql in practice

To go back to __GAE__, as you probably know it gives you a semi-free service to host your Java/Python/Php/Go applications. Upon registration, you are provided for free with a certain amount of bandwidth and DB transactions for free, but you can also buy payed plans and pay for what you actually use.
You can choose to use mySql for your DB, but as a default, your app will be connected to the datastore, which is based on google BigTable which in turn is built on Google File system, and the whole infrastructure is targeted to allow for seamless, maximum scalability, using a noSql approach.
This means, you don't have joins. Well, at least you didn't in the earliest days. Now you have the ability to do cross-entity queries, but there are some important limitations (see [here](https://developers.google.com/appengine/docs/java/datastore/transactions) for more) - mainly because, as mentioned, this is a concrete example of the tradeoff between performance and features: the datastore leans towards performance, so if you use it, it is implied you don't need a relations-oriented solution (if you do, you can use mySql and likely skip the rest of this post).

If you decide to use [MongoDB](http://www.mongodb.org) or [Redis](http://redis.io) with [NodeJS](http://nodejs.org) on [Heroku](http://heroku.com), for example, you are going to run into the exact same issues, so the problem is a general one, and I'll try to give a general solution.

###NoSql and joins

Let's assume you designed your application and, given its characteristics, you opted for a nosql solution. Then, either during the design phase or at some point during development, you realize that, while 99% of your traffic will not require cross-entities (cross-tables, to use a more SQL-ish terminology) queries, occasionally you'll need joins on two or more tables (perhaps for a rarely used but high-value feature, perhaps to compute monthly statistics). Once again, let's assume it isn't a critical feature - otherwise, you would need a different solution - and - without any loss in generality - only two tables are involved. The only solution is, you tun two different queries, and then, for each row of the first table, you look for the proper row in the second one (or for the proper rows, depending on the kind of join).

####Pseudo-join in a synchronous environment

In a synchronous environment, you then have to run the first query, then run the second one, then merge the two retrieved datasets together. And between each query, your program will have to wait for the request to be completed. To make the situation worse, let's say you have to preprocess each dataset before being able to merge the two of them together. So the flow of control becomes:

1. Start query 1
2. Query 1 returns data1
3. Call preProcessData(data1)
4. preProcessData returns
5. Start query 2
6. Query 2 returns data2
7. Call preProcessData(data2)
8. preProcessData returns
9. Merge the data1 and data2
10. Do something with the result of the merge
11. Continue with the rest of the flow

In pseudo-JavaScript:

{% highlight javascript %}
var data1 = queryDB_Synch(query1);
data1 = preProcessData(data1);
var data2 = queryDB_Synch(query2);
data2 = preProcessData(data1);
var result = mergeData(data1, data2);
doSomething(result);
__next_statement__
{% endhighlight %}

####Pseudo-join in an asynchronous environment

Since some of this calls are asynchronous or equivalently have an asynchronous version, you might want to switch to a different flow of execution. But, without using promises, what would you do? You'd use nested callbacks, so for example, in pseudo-JavaScript:

{% highlight javascript %}
queryDB_Asynch(query1, function (err1, data1) {
  data1 = preProcessData(data1);
  queryDB_Asynch(query2, function(err2, data2) {
    preProcessData(data2);
    var result = mergeData(data1, data2);
    doSomething(result);
  });
});
__next_statement__
{% endhighlight %}

where __doSomething__ is a callback performing the operations at point 10 of the synchronous workflow, and checks on the error flags are omitted for clarity.

The question is: this we gain any advantage with this transformation? The truth is, it depends. At the moment the only difference is that in the second version the rest of the flow is executed immediately after the first query is started, while in the first version it has to wait until all the requests are completed and the results merged. If we need this result to continue execution anyway, or if no other statement has to be executed, we gained no benefit at all.

###Exploiting asynchronicity - the wrong way

Where could we improve this? Well, for example, we notice that we don't need to wait for the first bunch of data to be processed to start the second query. Actually, we don't even need for the first query to be completed to start the second one!  What we _really_ want to do is:

1. Start the first query
2. Start the second query
3. continue execution of next statements
4. (A1) As soon as the first query returns, preProcess data1
5. (A2) As soon as the second query returns, preProcess data2
6. (B) When both calls to preProcess returns their result, call mergeData
7. (C) call doSomething on the result of mergeData

So, if for example we know that query2 requires more time than query1 to be completed, one might be tempeted to code up something like this:

{% highlight javascript %}
//DO NOT do this
queryDB_Asynch(query2, function(err2, data2) {
  preProcessData(data2);
  var result = mergeData(data1, data2);
  doSomething(result);
});
queryDB_Asynch(query1, function (err1, data1) {
  data1 = preProcessData(data1);
});
{% endhighlight %}

But this doesn't really work. Or, even worse, it could work some times, or most of the times, and then fail when you least expect it, throwing you in a endless painful nightimare before you can figure out what happens.
Moving the quickest query before the slowest won't help either:

{% highlight javascript %}
//DO NOT do this either
queryDB_Asynch(query1, function (err1, data1) {
  data1 = preProcessData(data1);
});
queryDB_Asynch(query2, function(err2, data2) {
  preProcessData(data2);
  var result = mergeData(data1, data2);
  doSomething(result);
});
{% endhighlight %}

The point is, there is no way you can tell which query will return first. Even if there is a huge difference in the size of the tables and/or in the number of results retrieved, when dealing with asynch calls, the behaviour is unpredictable, because a lot of issues could cause an unexpected latency.

####Exploiting asynchronicity - the wronger way

It might look like employing flag variables to check if the other branch has been completed is a good idea:

{% highlight javascript %}
//Definitely DO NOT do this EVER!
var queryCompleted_1 = false,
    queryCompleted_2 = false;
queryDB_Asynch(query1, function (err1, data1) {
  data1 = preProcessData(data1);
  if (queryCompleted_2) {
    var result = mergeData(data1, data2);
    doSomething(result);
  } else {
    queryCompleted_1 = true;
  }
});

queryDB_Asynch(query2, function(err2, data2) {
  data2 = preProcessData(data2);
  if (queryCompleted_1) {
    var result = mergeData(data1, data2);
    doSomething(result);
  } else {
    queryCompleted_2 = true;
  }
});
{% endhighlight %}

As you can perhaps have already guessed, it isn't! This is not only inefficient, not _DRY_, and quite frankly ugly, this is a potential recipe for disaster, because you expose yourself to a potential race condition.
As unlikely as it can be, since there is no way to ensure locks or atomicity, it can happen that between the execution of the if statement that checks queryCompleted_i in the first branch to complete the query and the next execution of the assignment in the relative else branch, the other branch reaches as well the same execution points, so that in that branch the if condition fails as well, and none of them ends up merging the results and calling doSomething. Since you don't control the order of execution of the instructions, you can't make any assumption.
And clearly a race condition is hardly ever an acceptable risk.

###Promises save the day

So, this is where promises and deferred comes into play.

If you know how promises and deferred work in JavaScript, or if you have read the posts linked above, you should know what we need here: while step _1 -> A1_ and _2 -> A2_ can be performed using callbacks, step _(A1 & A2) -> C_ needs a new way: the **_when_** method, which is specifically designed to synch parallel tasks!

In __jQuery__, _$.when()_ creates a new promise which will be resolved if both promises inside are resolved, or rejected if one of the promises fails. You can pass any number of arguments to _when_, and it takes even non-promise ones: they will be treated as a resolved promise.

One more caveat that needs to be mentioned before proceeding to the final version of our code: if _then()_ is passed a function which returns a promise object, the new promise will have the same behaviour as the returned promise; if, on the other hand, _then()_ is passed a function which returns a value, the value becomes the value of the new object.

So, all considered, we want to create two deferred , one for the first branch, and one for the second one, which return promises resolved only when the data retrieved from their queries is successfully preProcessed.

{% highlight javascript %}
var queryBranch_1 = $.Deferred(),
    queryBranch_2 = $.Deferred();

queryDB_Asynch(query1, function (err1, data1) {
  data1 = preProcessData(data1);
  queryBranch_1.resolve(data1);
});

queryDB_Asynch(query2, function(err2, data2) {
  data2 = preProcessData(data2);
  queryBranch_2.resolve(data2);
});

$.when(queryBranch_1.promise(), queryBranch_2.promise())
  .then(function(data1, data2) {
    var result = mergeData(data1, data2);
    doSomething(result);
});
{% endhighlight %}

Or, in the hypothesis that queryDB_Asynch returns a promise as well, and moving the code around a little bit:

{% highlight javascript %}
var queryBranch_1 = $.Deferred(),
    queryBranch_2 = $.Deferred();

queryDB_Asynch(query1)
  .then(function (data1) {
          queryBranch_1.resolve(preProcessData(data1));
  });

queryDB_Asynch(query2)
  .then(function (data2) {
          queryBranch_2.resolve(preProcessData(data2));
  });

$.when(queryBranch_1.promise(), queryBranch_2.promise())
  .then(function(data1, data2) {
    doSomething(mergeData(data1, data2));
});
{% endhighlight %}

Simple, elegant, efficient and race-free.

###Conclusions

Will see an example in a second. I just wanted to state a few considerations about the issue.
Well, the JavaScript-simulated JOIN was mainly an excuse to talk about neat tricks with promises, but, as I said earlier, there are scenarios where you can actually need it. I recently bumped into one of these, and I'm glad I had to work my way to this solution - so I thought it might be helpful to share it.
There is, of course, a penalty you have to pay: I tried running the same query as a native JOIN on a BigQuery DB, and as a JavaScript-simulated join: the native version is, as expected, much faster - the speedup is more than 10x!
So if you need to perform joins often and if it's a critical operation for your application, you might want to reconsider your DB solution, or at least you must take this performance tradeoff into account.

###Real code

Now, here it is a real-life example of simulated left join query in JavaScript, retrieving JSON data from a DB through ajax calls; this example uses promises a bit more extensively because takes authentication into account as well.

{% highlight javascript %}
/** @method leftJoin
  * @for dataloader
  *
  *    Simulate a left join query on the datataset, if the dataset doesn't allow join queries. Once the data is loaded, the result is passed to a callback
  * 
  * @param {Function} callback    The callback to be executed after data is loaded (on success). 
  *                                It can be any function taking data as second parameter and an error or null as first (node.js style):
  *                                for example, it can put the data on the console or use it to build a chart.
  * @param {String} leftTableQuery     The query to be run on the first table of the join.
  * @param {String} rightTableQuery The query to be run on the second table of the join.
  * @param {String} leftTableJoinField     The field of the left table to be used for the join.
  * @param {String} rightTableJoinField The field of the right table to be used for the join.
  * @param {Array} joinFields A list of the fields of the right table that are selected for the query (all of the left table ones will be).
  *    @param {Object} authParameters    The parameters to be passed to the oauth server.
  *    @param {Function} [adapter]    Optionally an adapter can be passed to process the data as retrieved from the server before returning it.
  * @return {Promise}    A promise that in the end will assume the value returned from callback once called on the result of the join query.
  *    @throw TypeError if the callback parameter or the query values aren't valid.
  */
function leftJoin (callback, authUrl, queryUrl, leftTableQuery, rightTableQuery, leftTableJoinField, rightTableJoinField, joinFields, authParameters, adapter) {
    "use strict";
    
    if (typeof callback !== "function" || callback.length < 1) {
        throw new TypeError("A callback function taking at least 1 argument (the json retrieved) is needed");
    }
    if (typeof rightTableQuery !== "string" || typeof leftTableQuery !== "string"
        || typeof leftTableJoinField !== "string" || typeof rightTableJoinField !== "string"
        || !$.isArray(joinFields)
        || typeof queryUrl !== "string" || typeof authUrl !== "string"){
                
                throw new TypeError("Invalid query parameters");
    }                            

    var leftTableProcessDeferred = $.Deferred(),
        rightTableProcessDeferred = $.Deferred(),

        onAuthToken = function (data) {
          var retrievalProcessDeferred = $.Deferred();
          //First query
          $.ajax({
              method: 'POST',
              url: queryUrl,
              data: JSON.stringify({'query' : rightTableQuery}),
              processData: false,
              headers: {'Content-Type': 'application/json', 'Authorization': 'Bearer ' + data.access_token}                            
          }).done(onRightTableQuerySuccess)
            .fail(function (jqXHR, textStatus) {
                    retrievalProcessDeferred.reject(jqXHR);
            });

          //Second query
          $.ajax({
              method: 'POST',
              url: queryUrl,
              data: JSON.stringify({'query': leftTableQuery}),
              processData: false,
              headers: {'Content-Type': 'application/json', 'Authorization': 'Bearer ' + data.access_token}                            
          }).done(onLeftTableQuerySuccess)
            .fail(function (jqXHR, textStatus) {
                    retrievalProcessDeferred.reject(jqXHR);
            });

          $.when(
              leftTableProcessDeferred,
              rightTableProcessDeferred
          ).then(function (leftTableData, rightTableData) {
            retrievalProcessDeferred.resolve(leftTableData, rightTableData);
          }).fail() {
                    retrievalProcessDeferred.reject();
          });
          return retrievalProcessDeferred.promise();  //Return a promise that will be resolved 
        },

        onRightTableQuerySuccess =  function (rightTableData) {
                                      //Records of the first table, indexed by the field to join (rightTableJoinField)
                                      var rightTableById = {},    
                                          n, i, t;

                                      if (typeof adapter === "function") {
                                          rightTableData = adapter(rightTableData);
                                      }

                                      //Process the transaction data, and then resolve the promise
                                      n = rightTableData.length;
                                      for (i = 0; i < n; i++) {
                                          t = rightTableData[i];
                                          if (t && typeof t[rightTableJoinField] !== undefined){
                                              rightTableById[t[rightTableJoinField]] = t;
                                          }
                                      }

                                      rightTableProcessDeferred.resolve(rightTableById);
                                  },
      
        onLeftTableQuerySuccess = function (leftTableData) {
                                      if (typeof adapter === "function") {
                                          leftTableData = adapter(leftTableData);
                                      } 
                                      leftTableProcessDeferred.resolve(leftTableData);
                                  },

        onDataProcessed = function (leftTableData, rightTableData) {
          var row, i, j, 
              m = joinFields.length, 
              n = leftTableData.length,
              id, 
              field;

          for (i = 0; i < n; i++) {
              row = leftTableData[i];
              id = row && row[leftTableJoinField];
              if (id) {
                  //Copy all joined fields
                  for (j = 0; j < m; j++){
                      field = joinFields[j];
                      row[field] = rightTableData[id] && rightTableData[id][field];
                  }
              }
          }

          return callback(null, leftTableData);  //onDataProcessedDeferred.promise();
        };     
    
  return $.ajax({
            method: 'POST',
            url: authUrl,
            data: authParameters,
            headers: {'Content-Type': 'application/x-www-form-urlencoded'}
        }
      ).then(onAuthToken)
      .then(function(l, r) {
        return onDataProcessed(l,r);
      })
      .fail(callback);    
}
{% endhighlight %}
