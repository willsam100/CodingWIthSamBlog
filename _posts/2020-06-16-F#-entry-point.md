---
title: 'F# EntryPoint, .NET Core and static exceptions'
date: 2020-06-28 21:00:00 +12:00
categories: [F#]
tags: [F#, .NETCore, AWS, Lambda]
description: F# static methods and .NET Core EntryPoint
# image: /assets/img/2020-06-13-client.png
published: false
---

## Motivation 

F# has very few situations where the code execution is bizarre. We tend to love F# because of this reason, reading code is almost the same thing as executing it. This post is on one of those bizarre cases. 

To be evident in the several years that I have using F#, I have only come across 2 cases. This is the primary reason why I enjoy writing F# so much. In almost all cases, the language is predictable. 

## A Suprise in F#

This is the exception that occurs: 

> System.InvalidOperationException : The static initialization of a file or type resulted in static data being accessed recursively before it was fully initialized.

Here is the code:

```fsharp 
namespace LambdaEntryPoint
open Amazon.Lambda.Core
open Amazon.Lambda.APIGatewayEvents
open System.Net

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[<assembly: LambdaSerializer(typeof<Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer>)>]
()

type Functions() =
    static let v = 42
    
    member __.Get (request: APIGatewayProxyRequest) (context: ILambdaContext) =
        sprintf "Request: %s" request.Path
        |> context.Logger.LogLine

        printfn "%d" v

        APIGatewayProxyResponse(
            StatusCode = int HttpStatusCode.OK,
            Body = "Hello AWS Serverless",
            Headers = dict [ ("Content-Type", "text/plain") ]
        )
```

This code was created by running `dotnet new serverless.EmptyServerless -lang F# -n LambdaEntryPoint`, and then adding the static value `v` along with the print statement. 

The project type must also be set to `Console Application`, i.e. the following must be added to the fsproj file `<OutputType>Exe</OutputType>`.

To trigger the exception, run the out-of-box unit test. 

## Cause

It turns out that static initialization is triggered by the attribute `EntryPoint` if the project is a console application. Given the project was created as a class library, and then manually converted to console application, an entry point was not added. Static initialization does not run. 

To be fair, this is quite rare. Most projects will either be a class library or run as a console application (i.e. have an `EntryPoint`). In this case, working with AWS Lambda, a series of changes across a team lead to this situation. 

### Solutions

We're working with F# here, so all of the solutions are going to be pretty straightforward. The critical point is to make sure that at least one of them is present.

### Make sure there is an EntryPoint

If the project is a console application (exe), then make sure `[<EntryPoint>]` is present in the project. If the library is ever changed from a class library to a console application, the next step should be to add an `EntryPoint`. 

### Move to a Standard library

The above case is avoided with the default template provided by AWS because the project is a class library. Changing it back to a class library would also address the solution. 

The AWS lambda invocation model calls directly into the library, so there really is no need for it to be a console application. 

### Avoid static methods 

This is probably the least desirable solution, but it would still solve the problem above. Removing the `static` keyword means the code runs fine; an excellent trick to remember if you're in a bind and need to get the software operating.

## Key Takeaway

Keep an eye out for `static` usage in a console project without an `[<EntryPoint>]` - it might happen to your team, the result is quite exceptional. 

