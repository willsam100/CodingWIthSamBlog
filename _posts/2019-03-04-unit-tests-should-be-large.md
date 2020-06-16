---
layout: post
title: Unit tests should be large, not small
date: 2019-03-04 19:00:14.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Testing]
tags: [F#, Functional-Programming, PureFunctions, Testing, Unit-Tests]
meta:
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1591203474'
  _yoast_wpseo_primary_category: '28'
  _yoast_wpseo_content_score: '30'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  _yoast_wpseo_metadesc: Unit that don't catch regressions are useless. Unit tests
    should cover pure functions. The pure function should be as large (code coverage)
    as possible.
  mashsb_og_image: '547'
  mashsb_og_title: Unit tests should be large, not small
  mashsb_og_description: 'Unit that don''t catch regressions are useless. Unit tests
    should cover pure functions. The pure function should be as large as possible. '
  mashsb_shares: '0'
# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
permalink: "/2019/03/04/unit-tests-should-be-large/"
---
If a unit test does not catch a regression, especially after refactoring, it is pointless. Small unit tests typically don't catch regressions. If the code base will not live for very long (eg proof-of-concept) then manually test.

When refactoring, or adding new features, the structure of the project typically changes in non-trivial ways. Small unit tests typically break. If a unit test breaks then it serves little purpose, as now you need to test again (re-writing the test counts as testing again). The idea behind unit tests is to make code changes without breaking existing code. The tests should not require changing.

## What does large mean?
A large unit test does not mean the test code itself is large, but rather that the coverage of the test is large. The test should cover functions/classes working together from one external dependency to another. External dependencies define the upper bound on how large a unit test should be. Unit tests should not have any external dependencies.

External dependencies include APIs, databases, OS/Phone APIs etc. A library that does a data transform, is not an external dependency (since we could write the code ourselves). eg JSON conversion. These libraries should be considered internal.

## A large unit test should cover a pure function
By making our unit tests cover a large surface of our code base, we need to make our code base reliable and predictable. **Pure functions are predictable**. Given the same data, they produce the same output. For more on this here is why we should use [pure functions not interfaces]({{ "/2019/02/22/interfaces-vs-pure-functions/" | relative_url }}). Modelling our software from external dependency to another means that we build pure functions. This pushes nasty things like state/mutation to the edge of the system, where it is easier to control.

Let's make this concrete, with an example we can walk through. Here is a pure function that needs testing <code>parseDomainType</code> assuming we are building a notes app. Sample code will be in F# (because it is awesome), though the concepts can be applied to any language be that C# or Java.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l"> 1: </span>
<span class="l"> 2: </span>
<span class="l"> 3: </span>
<span class="l"> 4: </span>
<span class="l"> 5: </span>
<span class="l"> 6: </span>
<span class="l"> 7: </span>
<span class="l"> 8: </span>
<span class="l"> 9: </span>
<span class="l">10: </span>
<span class="l">11: </span>
<span class="l">12: </span>
<span class="l">13: </span>
<span class="l">14: </span>
<span class="l">15: </span>
<span class="l">16: </span>
<span class="l">17: </span>
<span class="l">18: </span>
<span class="l">19: </span>
<span class="l">20: </span>
<span class="l">21: </span>
<span class="l">22: </span>
<span class="l">23: </span>
<span class="l">24: </span>
<span class="l">25: </span>
<span class="l">26: </span>
<span class="l">27: </span>
<span class="l">28: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// A DTO object that has correct equality</span>
<span class="k">type</span> <span class="rt">RestApiResponse</span> <span class="o">=</span> <span class="pn">{</span>
    <span class="id">StatusCode</span><span class="pn">:</span><span class="vt">int</span>
    <span class="id">Reponse</span><span class="pn">:</span><span class="rt">string</span>
    <span class="id">Headers</span><span class="pn">:</span> <span class="rt">Map</span><span class="pn">&lt;</span><span class="rt">string</span><span class="pn">,</span><span class="rt">string</span><span class="pn">&gt;</span>
<span class="pn">}</span>

<span class="c">// An enum that can hold data on each case (Called a discriminated union)</span>
<span class="k">type</span> <span class="rt">RestApiError</span> <span class="o">=</span> 
<span class="pn">|</span> <span class="uc">ServerError</span> <span class="k">of</span> <span class="id">response</span><span class="pn">:</span><span class="rt">string</span>
<span class="pn">|</span> <span class="uc">ClientError</span> <span class="k">of</span> <span class="id">response</span><span class="pn">:</span><span class="rt">string</span>
<span class="pn">|</span> <span class="uc">MalformedResponse</span> <span class="k">of</span> <span class="id">error</span><span class="pn">:</span><span class="rt">exn</span>

<span class="c">// Our target type - yep we're building a notes apps :) </span>
<span class="c">// A DTO object that has correct equality</span>
<span class="k">type</span> <span class="rt">Note</span> <span class="o">=</span> <span class="pn">{</span>
    <span class="id">Created</span><span class="pn">:</span> <span class="id">DateTime</span>
    <span class="id">Message</span><span class="pn">:</span> <span class="rt">string</span>
<span class="pn">}</span>

<span class="c">// This is only showing the types. Input is a RestApiResponse, while the output is a type</span>
<span class="c">// the could be success some type 'a, or an error of type RestApiError</span>
<span class="id">parseDomainType</span><span class="pn">:</span> <span class="id">RestApiResponse</span> <span class="k">-&gt;</span> <span class="id">Result</span><span class="pn">&lt;</span><span class="id">'</span><span class="id">a</span><span class="pn">,</span> <span class="id">RestApiError</span><span class="pn">&gt;</span>

<span class="c">// usage </span>
<span class="k">let</span> <span class="id">response</span><span class="pn">:</span> <span class="id">RestApiResponse</span> <span class="o">=</span> <span class="c">// some web call here</span>
<span class="k">let</span> <span class="id">notes</span> <span class="o">=</span> <span class="id">parseDomainType</span><span class="pn">&lt;</span><span class="id">List</span><span class="pn">&lt;</span><span class="id">Note</span><span class="pn">&gt;</span><span class="pn">&gt;</span> <span class="id">response</span>
<span class="c">// show notes on mobile app UI</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Given this is a pure function, it is not particularly important how this function works. I will post links below GitHub with a full implementation.
To find the inputs and outputs of the API, all that was required was to think about the edges of the system. One edge was the API to download the notes. The other edge was the UI. These are both external dependencies. We simply model the correct types for them. Considering an API, it is clear that it might fail, so the response may not have valid data. This highlights that our pure function 'parseDomainType` must return a result that captures the notion of failure.

## Include internal dependencies
An internal dependency is simply a function/method/class that does a code transformation. It could be in your code base, or it could be a library. As long as the code does not do anything with the outside world, the unit test should cover it. Said another way, internal dependencies should be pure functions not interfaces]({{ "/2019/02/22/interfaces-vs-pure-functions/" | relative_url }}). Pure functions can be glued together (that is why they are some awesome), so our main function under test <code>parseDomainType</code>, is built out of small pure functions. Some of those functions are business logic. and others are libraries.

The fact that <code>parseDomainType</code> may use a library is simply an implementation detail. Swapping it out should not change the behaviour. The ability to swap out pure functions (and have the types still line up) has a name - referential transparency. It's what gives us the confidence to change our software and know (simply by compiling our code and that we're using pure functions) that we haven't broken anything.

Our unit tests confirm that our code meets some acceptance criteria. ie just because our code compiles and doesn't throw exceptions, does not guarantee that it does the right thing according to the acceptance criteria.

## Small tests might help during development, not after
When writing the domain logic for my app, I prefer to use a script file and a REPL. Many languages have these including F#, Python, C#, and Java 9+ (though some are easier to use than others). This makes it really easy to build code, and check it works. It' a substitute for small tests.

If your language does not have these tools, small tests can be used to prove out details of how single function works. Once the small test passes, write a large test, and delete the small test. Commit only the tests that are large.

Sometime later when the code needs to be changed, the small test will be a distraction. A false positive. Delete it now to improve productivity.
A few small tests (over some very important logic) are fine, but too many small unit tests become noise and hide the large useful unit tests.

## Small tests are harder to name
Naming a large unit test (remember large means the amount of code coverage, not the count of lines in the test) are easier to name as the path through the code base is more high level. More concrete. Small tests, however, focus more on the details and are less low level. They are testing a tiny piece that now requires a precise name, for a code piece that is not clear what it does when considered outside of the context of the large function eg <code>parseDomainType</code>.

## A worked example
It's time for an example to make this all a little more concrete. As already stated we have our pure function that converts an HTTP response into a list of notes for our notes app.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">parseDomainType</span><span class="pn">&lt;</span><span class="id">'</span><span class="id">a</span><span class="pn">&gt;</span> <span class="pn">(</span><span class="id">response</span><span class="pn">:</span> <span class="id">RestApiResponse</span><span class="pn">)</span><span class="pn">:</span> <span class="id">Result</span><span class="pn">&lt;</span><span class="id">'</span><span class="id">a</span><span class="pn">,</span> <span class="id">RestApiError</span><span class="pn">&gt;</span> <span class="o">=</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Our first test might be the happy path:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l"> 1: </span>
<span class="l"> 2: </span>
<span class="l"> 3: </span>
<span class="l"> 4: </span>
<span class="l"> 5: </span>
<span class="l"> 6: </span>
<span class="l"> 7: </span>
<span class="l"> 8: </span>
<span class="l"> 9: </span>
<span class="l">10: </span>
<span class="l">11: </span>
<span class="l">12: </span>
<span class="l">13: </span>
<span class="l">14: </span>
<span class="l">15: </span>
<span class="l">16: </span>
<span class="l">17: </span>
<span class="l">18: </span>
<span class="l">19: </span>
<span class="l">20: </span>
<span class="l">21: </span>
<span class="l">22: </span>
<span class="l">23: </span>
<span class="l">24: </span>
<span class="l">25: </span>
<span class="l">26: </span>
<span class="l">27: </span>
<span class="l">28: </span>
<span class="l">29: </span>
<span class="l">30: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">open</span> <span class="id">NUnit</span><span class="pn">.</span><span class="id">Framework</span>
<span class="k">open</span> <span class="id">System</span>

<span class="pn">[&lt;</span><span class="id">TestFixture</span><span class="pn">&gt;]</span>
<span class="k">type</span> <span class="id">NoteTests</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">open</span> <span class="id">FsUnit</span>

    <span class="pn">[&lt;</span><span class="id">Test</span><span class="pn">&gt;]</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">``Given a response parseDomainType can parse a list of notes``</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 

        <span class="c">// Create the input data for our test</span>
        <span class="k">let</span> <span class="id">response</span> <span class="o">=</span> <span class="pn">{</span>
            <span class="id">NotesService</span><span class="pn">.</span><span class="id">StatusCode</span> <span class="o">=</span> <span class="n">200</span>
            <span class="id">NotesService</span><span class="pn">.</span><span class="id">Body</span> <span class="o">=</span> 
                <span class="s">"""[{</span>
<span class="s">                        "Created": "2019-02-02",</span>
<span class="s">                        "Message": "First note"</span>
<span class="s">                    }] """</span>
            <span class="id">NotesService</span><span class="pn">.</span><span class="id">Headers</span> <span class="o">=</span> <span class="id">Map</span><span class="pn">.</span><span class="id">empty</span>
        <span class="pn">}</span>

        <span class="c">// act - run the pure function with the pure data</span>
        <span class="k">let</span> <span class="id">notes</span> <span class="o">=</span> <span class="id">NotesService</span><span class="pn">.</span><span class="id">parseDomainType</span><span class="pn">&lt;</span><span class="id">NotesService</span><span class="pn">.</span><span class="id">Note</span> <span class="id">list</span><span class="pn">&gt;</span> <span class="id">response</span>

        <span class="c">// assert on the result</span>
        <span class="k">match</span> <span class="id">notes</span> <span class="k">with</span> 
        <span class="pn">|</span> <span class="id">Ok</span> <span class="id">notes</span> <span class="k">-&gt;</span> 
            <span class="k">let</span> <span class="id">expected</span> <span class="o">=</span> <span class="pn">{</span> <span class="id">NotesService</span><span class="pn">.</span><span class="id">Created</span> <span class="o">=</span> <span class="id">DateTime</span><span class="pn">(</span><span class="n">2019</span><span class="pn">,</span> <span class="n">02</span><span class="pn">,</span> <span class="n">02</span><span class="pn">,</span> <span class="n">00</span><span class="pn">,</span> <span class="n">00</span><span class="pn">,</span> <span class="n">00</span><span class="pn">,</span> <span class="id">DateTimeKind</span><span class="pn">.</span><span class="id">Utc</span><span class="pn">)</span><span class="pn">;</span> <span class="id">NotesService</span><span class="pn">.</span><span class="id">Message</span> <span class="o">=</span> <span class="s">"First note"</span> <span class="pn">}</span>
            <span class="id">notes</span> <span class="o">|&gt;</span> <span class="id">List</span><span class="pn">.</span><span class="id">head</span> <span class="o">|&gt;</span> <span class="id">should</span> <span class="id">equal</span> <span class="id">expected</span>
        <span class="pn">|</span> <span class="id">Error</span> <span class="id">e</span> <span class="k">-&gt;</span> <span class="id">failwithf</span> <span class="s">"Expected Ok notes but got: %A"</span> <span class="id">e</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
In the code above NUnit is used. A class is defined to hold the tests. A library, FsUnit is also used for assertions. A method is defined, that takes advantage of F#'s naming feature so the name of the method can contain spaces. The test then creates the input data and calls the function under test with that data. Finally, the test pattern matches on the result (since the function defines that it could fail). If the response is an error the tests fail (by throwing an appropriate message). If the parse succeeds, the test then checks the objects are correct which is easy with F#'s record types as they implement equality on all fields for us.

The above test is not very long but tests quite a lot of our application. First shows the dev reading the test what the raw expected JSON should be. This is very helpful, as it clearly defines how any parsing logic should work. It's also useful if the backend is not using the exact same tech stack, and they need to see exactly what the output should be. Finally, it makes it easy to upgrade or change the parsing library, as this will check that the behaviour has not changed. The response object requires all fields to be defined so this makes it clear when to expect success or failure.

What remains now is to add a set of tests varying the input with the expected output. These include changing the status-code, the response body and the headers. When all of these tests are added, there is quite a bit of repetition in the tests. A small amount of code clean that up, so the above tests could look as follows:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
<span class="l">4: </span>
<span class="l">5: </span>
<span class="l">6: </span>
<span class="l">7: </span>
<span class="l">8: </span>
<span class="l">9: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="pn">[&lt;</span><span class="id">Test</span><span class="pn">&gt;]</span>
<span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">``Given a response parseDomainType can parse a list of notes``</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="id">successfulResponseWithBody</span> <span class="s">"""[{"Created":"2019-02-02","Message":"First note"}]"""</span>
    <span class="o">|&gt;</span> <span class="id">NotesService</span><span class="pn">.</span><span class="id">parseDomainType</span><span class="pn">&lt;</span><span class="id">NotesService</span><span class="pn">.</span><span class="id">Note</span> <span class="id">list</span><span class="pn">&gt;</span>
    <span class="o">|&gt;</span> <span class="id">expectOk</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">result</span> <span class="k">-&gt;</span> 
    
        <span class="k">let</span> <span class="id">expected</span> <span class="o">=</span> <span class="pn">{</span> <span class="id">NotesService</span><span class="pn">.</span><span class="id">Created</span> <span class="o">=</span> <span class="id">DateTime</span><span class="pn">(</span><span class="n">2019</span><span class="pn">,</span> <span class="n">02</span><span class="pn">,</span> <span class="n">02</span><span class="pn">)</span><span class="pn">;</span> <span class="id">NotesService</span><span class="pn">.</span><span class="id">Message</span> <span class="o">=</span> <span class="s">"First note"</span> <span class="pn">}</span>
        <span class="id">result</span> <span class="o">|&gt;</span> <span class="id">List</span><span class="pn">.</span><span class="id">head</span> <span class="o">|&gt;</span> <span class="id">should</span> <span class="id">equal</span> <span class="id">expected</span>
        <span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
For such a small amount of code, there is quite a lot being tested here. This test is very clear about what it is testing too.

## Bugs won't be in the business logic
As can be seen in the example above, it is very easy to create unit tests and assert the code is correct. The code that causes bugs/exceptions (external dependencies) will now be pushed to the edge of the system, for an MVVM solution that could be the view model. The search space is now much smaller when problems do arise, as we know that our business logic and associated libraries are well tested, so the search should begin at the edge; the view models.

## Refactoring
This is one of the biggest advantages of creating large tests over pure functions. The internals of the library can be changed without breaking the tests. There are not mocks, so they can't break. The test is not aware of any internal libraries that may be changed or upgraded (they are an implementation detail).

Finally, because these tests are easy to create, there should be a good leave of code coverage - refactor fearlessly. With confidence in our tests, we can make constant or large changes to the internals of the function knowing that nothing will be broken. This means the code base will remain clean over time and as we all know, clean code is much nicer than bad code.

## Taking Action
If you are not familiar with pure functions then learn about those now:

<a>Pure functions</a>

[pure functions not interfaces]({{ "/2019/02/22/interfaces-vs-pure-functions/" | relative_url }})
Read up on the full implementation for this post here:

<a title="https://gist.github.com/willsam100/50e47ead0f5598875e2052de0acb25bf">parseDomainType as a pure function</a>

<a title="https://gist.github.com/willsam100/2a449691570dd5f80b38204b50c240f7">testing parseDomainType with large tests</a>
Practice in a small demo app, using a domain that you understand well. These concepts can be hard at first to understand. By starting from a clean slate with a known problem, it is possible to make progress.
Happy [type safe] coding

CodingWithSam