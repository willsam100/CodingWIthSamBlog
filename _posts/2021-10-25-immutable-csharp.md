---
title: Immutable C#
date: 2021-10-25 21:00:00 +12:00
categories: [C#]
tags: [C#, Functional Programming]
description: Modern C# versions make it very easy to write immutable C#
published: true
image: /assets/img/2021-10-25-immutable-csharp.jpg
---

# Immutable CSharp

Getting software right is hard. Any tool that helps us move in that direction is going to be a good choice. One clear challenge is managing things that change. Just like goto is considered harmful, mutating objects should not be the default choice. In C#, it's not possible to make immutable objects the default choice. 

## A contrived business case

Fetch all banking transactions from the database and calculate the average price. Tax should be ignored. Transactions in March and earlier do not have tax, while transactions after March are tax inclusive. 


## A conventional approach

First, we need a DTO to hold the data as well as a database service. 

    public class Transaction 
    {
        public DateTime PurchaseDate { get; set; }
        public decimal Amount { get; set; }
    }

    public interface IDatabase
    {
        public List<Transaction> GetTransactions(int userId);
    }

Now we can use a class to glue everything together. 

    public class MyService
    {
        public readonly IDatabase _database;

        public MyService(IDatabase database)
        {
            _database = database;
        }

        private static int ToCents(decimal amount) => (int) (amount * 100);

        private static void RemoveTax(Transaction t)
        {
            if (t.PurchaseDate.Month >= 3)
                return;

            t.Amount = t.Amount - (t.Amount * .15m / 1.15m);
        }
        
        public double AveragePurchasePrice(int userId)
        {
            try
            {
                var transactions = _database.GetTransactions(userId);
                foreach (var transaction in transactions)
                {
                    RemoveTax(transaction);
                }
                
                var total = (double) transactions.Sum(x => ToCents(x.Amount));
                return Math.Round(total / transactions.Count, 2);
            }
            catch (Exception e)
            {
                // handle bad stuff
                throw;
            }
        }
    }


A few parts to call out is the mutation that occurs when removing the GST. Additionally, the error handling for the DB is not particularly useful. Now for some improvements. 

## C# 9 records and init properties 


We can change the `Transaction` DTO to be immutable. The record has a basic form of equality along with an updated syntax, and the `init` properties mean that once set at creation, the values cannot be changed.

    public record Transaction 
    {
        public DateTime PurchaseDate { get; init; }
        public decimal Amount { get; init; }
    }


The `RemoveTax` method will fail to compile. Here is an improved version:

        private static Transaction RemoveTax(Transaction t)
        {
            if (t.PurchaseDate.Month >= 3)
                return t;

            return t with
            {
                Amount = t.Amount - (t.Amount * .15m / 1.15m)
            };
        }

The record syntax makes this very easy to create a copy and update the value. Before C# 9, this would have required a lot more code. 

Given that `Transaction` is now immutable, we need to return a copy (as can be seen in the method now). This means we also need to update `AveragePurchasePrice`. In the previous version, we had `foreach` loop. We can do better. 

    public double AveragePurchasePrice(int userId)
    {
        try
        {
            var transactions = _database.GetTransactions(userId);
            var total = (double) transactions.Select(RemoveTax).Sum(x => ToCents(x.Amount));
            return Math.Round(total / transactions.Count, 2);
        }
        catch (Exception e)
        {
            // handle bad stuff
            throw;
        }
    }

The `foreach` loop has been replaced with a much cleaner `Select` statement. This provides the added benefit of reducing a variable (the result of the `foreach`), since naming is hard. the `total` is a chained expression. 


## Improved error handling

Our latest approach improved the business logic, but it didn't address any of the error handling. The solution does a rather poor job at error handling. More importantly, the tools are not helping us at all. 

Add the following library to the project (this is the only library we need to add): `CSharpFunctionalExtensions`

We can now update our `IDatabase` to be more explicit about errors:

    public abstract record Error
    {
        public string Message { get; init; }
    }

    public record DatabaseError : Error
    {
        public Exception Exception { get; init; }
    }

    public interface IDatabase
    {
        public Result<List<Transaction>, Error> GetTransactions(int userId);
    }


We now have a clear error type (using the `record` type), and our `GetTransactions` method is very clear that it might fail. It provides a signal that we should probably handle that failure case. 

The code to convert an `Exception` into a result is not shown here. It is quite simple and could be added to the implementation of the `IDatabase` or added to another class to wrap the underlying implementation if the underlying implementation can not be changed. 

The code does not compile now in `AveragePurchasePrice`. Here is a very basic way to handle the error:

    public double AveragePurchasePrice(int userId)
    {
        try
        {
            var transactions = _database.GetTransactions(userId)
                .Match(x => x, _ => new List<Transaction>());
            var total = (double) transactions.Select(RemoveTax).Sum(x => ToCents(x.Amount));
            return Math.Round(total / transactions.Count, 2);
        }
        catch (Exception e)
        {
            // handle bad stuff
            throw;
        }
    }

This is better, as the code is specific about the database errors. Given this is business code, it would be a good time to consult the ticket/PO/PM for how to fail when this error occurs. To get the code compiling we can return an empty list. 

Let's do better though!

# Reporting errors

    public double AveragePurchasePrice(int userId)
    {
        try
        {
            return _database.GetTransactions(userId)
                .Map(transactions =>
                {
                    var total = (double) transactions.Select(RemoveTax).Sum(x => ToCents(x.Amount));
                    return Math.Round(total / transactions.Count, 2);
                })
                .Match(x => x, error =>
                {
                    if (error is DatabaseError dbError)
                    {
                        Console.WriteLine($"There was database error: ${dbError.Message}\n{dbError.Exception}");
                    }

                    return -69;
                });
        }
        catch (Exception e)
        {
            // handle bad stuff
            throw;
        }
    }

This is quite a change! We can now `Map` (the same as `Select` but for the `Result` type) over the transactions. The `Map` is a nice way to capture and contain the business logic to be applied to the transactions. 
The `Match` follows the `Map`, and for the happy path, we return the transactions. When there is an error, the code checks if the error is a known error case. We log a message in that case. For all errors, we return a default value. 

# Handling Business Case Errors

The code is in a much better place. Sadly, we have another bug that we didn't handle very well. Customers that don't have any transactions were seeing an error. It turns out that for an average, it is not possible to divide by 0. 
This is the offending line:
    
    return Math.Round(total / transactions.Count, 2);


The code should capture this, and aim to provide hints. Adding in explicit checks might be one way, let's ignore that for now and take another approach:

    private static Result<double, Error> Divide(int top, int bottom) =>
        bottom == 0
            ? Result.Failure<double, Error>(new MathError {Message = $"Attempted to divide by 0 {top}"})
            : top / bottom;

We have our own `Divide` that is much explicit about what it can and can't do. The default `/` in C# is clearly over promising what it can do. Now we update the broken line:

    return Math.Round(Divide(total, transactions.Count), 2);

Sadly this doesn't quite work. We get an error message saying an overload can't be found. `Math.Round` takes in numeric value and `Divide` is returning an error. The compiler is helping us!
Let's try to use `Map` on the result of `Divide`:

    return Divide(total, transactions.Count).Map(x => Math.Round(x, 2));

This is progress, but now there is another, something about Result and double! Again, the compiler is giving us feedback about our error handling. 

Here is the [failing] full code:

    return _database.GetTransactions(userId)
        .Map(transactions =>
        {
            var total = transactions.Select(RemoveTax).Sum(x => ToCents(x.Amount));
            return Divide(total, transactions.Count).Map(x => Math.Round(x, 2));
        })
        .Match(x => x, error =>
        {
            if (error is DatabaseError dbError)
            {
                Console.WriteLine($"There was database error: ${dbError.Message}\n{dbError.Exception}");
            }

            return -69;
        });
        
The error is telling us, that after fetching the transactions, we said that we would `Map` over them. `Map` is our way of saying 'this operation will always pass'. With our new `Divide` method, that is no longer true. To fix the error, replace `Map` with `Bind`. 

    public double AveragePurchasePrice(int userId)
    {
        return _database.GetTransactions(userId)
            .Bind(transactions =>
            {
                var total = transactions.Select(RemoveTax).Sum(x => ToCents(x.Amount));
                return Divide(total, transactions.Count).Map(x => Math.Round(x, 2));
            })
            .Match(x => x, error =>
            {
                if (error is DatabaseError dbError)
                {
                    Console.WriteLine($"There was database error: ${dbError.Message}\n{dbError.Exception}");
                }
                
                // Did you spot the bug here?

                return -69;
            });
    }
    

`AveragePurchasePrice` should now compile, and our code won't throw any exceptions anymore. This code is missing a few things still. 

# Ennumerating errors

One downside of the way we captured the errors was that the compiler was not able to help us when we added more errors. We can move to pattern matching, but this is still a limitation in C# (F# can warn for missing cases when pattern matching)

    .Match(x => x, error =>
    {
        Console.WriteLine(error switch
        {
            null => "Not a good outcome - let's hope for C# 20",
            DatabaseError dbError => $"There was database error: ${dbError.Message}\n{dbError.Exception}",
            MathError mathError => $"Math Erorr {mathError.Message}",
            _ => $"Random Error:{error.Message} of type {error.GetType()}",
        });

        if (error is MathError)
            return 0;
        
        return -69;
    });

The updated error handling can now provide detailed logging for each error. We are also able to customize the response based on the type of error. This has the advantage of consolidating all the error handling code to one place. 

# Code clean up

Quite a few changes have been made. Assuming we have a unit test, it's now possible to go through and clean up the code a bit more. 

    public double AveragePurchasePrice(int userId)
    {
        return _database.GetTransactions(userId)
            .Map(ts => ts.Select(x => ToCents(RemoveTax(x).Amount)).ToList())
            .Map(ts => (Total: ts.Sum(), ts.Count))
            .Bind(tuple =>
            {
                return Divide(tuple.Total, tuple.Count).Map(x => Math.Round(x, 2));
            })
            .Match(x => x, error =>
            {
                Console.WriteLine(error switch
                {
                    null => "Not a good outcome - let's hope for C# 20",
                    DatabaseError dbError => $"There was database error: ${dbError.Message}\n{dbError.Exception}",
                    MathError mathError => throw new NotImplementedException(),
                    _ => $"Random Error:{error.Message} of type {error.GetType()}",
                });
                
                if (error is MathError)
                    return 0;
                
                return -69;
            });
    }

Each line can be pulled out to its own `Map` statement to make it easier to read. With the new tuple syntax in C#, the functions can pass around data with ease.

The `try catch` should not be needed as all exceptions should already be captured using the `Result` type. 

And for the final step - swap out to expressions. Here is all the code:

    public record Transaction 
    {
        public DateTime PurchaseDate { get; init; }
        public decimal Amount { get; init; }
    }

    public abstract record Error
    {
        public string Message { get; init; }
    }

    public record DatabaseError : Error
    {
        public Exception Exception { get; init; }
    }

    public record MathError : Error { }

    public interface IDatabase
    {
        public Result<List<Transaction>, Error>  GetTransactions(int userId);
    }

    public class MyServiceTwo
    {
        private readonly IDatabase _database;

        public MyServiceTwo(IDatabase database)
        {
            _database = database;
        }

        private static int ToCents(decimal amount) => (int) (amount * 100);
        
        private static Transaction RemoveTax(Transaction t) =>
            t.PurchaseDate.Month >= 3
                ? t
                : t with { Amount = t.Amount - (t.Amount * .15m / 1.15m) };

    public double AveragePurchasePrice(int userId) =>
        _database.GetTransactions(userId)
            .Map(ts =>
                ts
                    .Select(RemoveTax)
                    .Select(x => ToCents(x.Amount))
                    .ToList())
            .Map(ts => (Total: ts.Sum(), ts.Count))
            .Bind(tuple => Divide(tuple.Total, tuple.Count))
            .Map(x => Math.Round(x, 2))
            .Match(x => x, error =>
            {
                Console.WriteLine(error switch
                {
                    null => "Not a good outcome - let's hope for C# 20",
                    DatabaseError dbError => $"There was database error: ${dbError.Message}\n{dbError.Exception}",
                    MathError mathError => throw new NotImplementedException(),
                    _ => $"Random Error:{error.Message} of type {error.GetType()}",
                });
                
                if (error is MathError)
                    return 0;
                
                return -69;
            });

We have done the impossible; removed most of the curly braces from C#!

For this contrived example, quite a few changes were made that have made the code much easier to debug, read, and handle errors. `AveragePurchasePrice` can be read from top to bottom, one line at a time. 

Each line of `AveragePurchasePrice` is very clear about what it does. Error handling is very explicit but with a concise syntax. When this code needs to change in the future, the compiler can help. Additionally, each part is self-contained. 

e.g. the error handling code is nicely contained in one place. 



# strech goal

For a syntax change, there is one other variation that can be made to the above example. 
    
    public static class TransactionDomain
    {
        public static int ToCents(this decimal amount) => (int) (amount * 100);

        public static Transaction RemoveTax(this Transaction t) =>
            t.PurchaseDate.Month >= 3
                ? t
                : t with { Amount = t.Amount - (t.Amount * .15m / 1.15m) };
    }

    public class MyServiceTwo
    {
        private readonly IDatabase _database;

        public MyServiceTwo(IDatabase database)
        {
            _database = database;
        }
        
        private static Transaction RemoveTax(Transaction t) =>
            t.PurchaseDate.Month >= 3
                ? t
                : t with { Amount = t.Amount * 1.15m };

        private static Result<double, Error> Divide(int top, int bottom) =>
            bottom == 0
                ? Result.Failure<double, Error>(new MathError {Message = $"Attempted to divide by 0 {top}"})
                : top / bottom;
        
        public double AveragePurchasePrice(int userId) =>
            _database.GetTransactions(userId)
                .Map(ts =>
                    ts
                        .Select(x => x.RemoveTax().Amount.ToCents())
                        .ToList())
                .Map(ts => (Total: ts.Sum(), ts.Count))
                .Bind(tuple => Divide(tuple.Total, tuple.Count).Map(x => Math.Round(x, 2)))
                .Match(x => x, error =>
                {
                    Console.WriteLine(error switch
                    {
                        null => "Not a good outcome - let's hope for C# 20",
                        DatabaseError dbError => $"There was database error: ${dbError.Message}\n{dbError.Exception}",
                        MathError mathError => throw new NotImplementedException(),
                        _ => $"Random Error:{error.Message} of type {error.GetType()}",
                    });
                    
                    return -69;
                });
    }

In this alternative version, the business logic has been pulled to a separate class. The class can be `static` because it should not contain any state. Unit tests would be easy to write. Additionally, the methods are implemented as extension methods. 

`AveragePurchasePrice` can now leverage the business class, and syntax is more declarative: `.Select(x => x.RemoveTax().Amount.ToCents())`


## What did I miss or misunderstand? leave a comment below!
