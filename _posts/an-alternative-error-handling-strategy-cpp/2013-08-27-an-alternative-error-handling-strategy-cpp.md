---
layout: post
title: An alternative error handling strategy for C++
description: We treat three historical ways to handle errors,
  and consider a functional approach using monades to handle error in C++.
  We provide a C++ implementation unsing templates and discuss the pros and cons.
author: Jérémy Cochoy
date: 2013-08-27 + 0100
categories: programming C++ monads language software script
lang: en
...

## Error handling

This not so short post is dedicated to a subject that may interest many
programers: error handling.

Error handling is the "dark side" of programming. It is both the heart of
real world applications, and the dirty stuff you would like to avoid.

Since the C years, I know three most common ways of handling errors.

## Do it C-style: return code

'C-Style' handling is the easiest way to do it, but it isn't completely satisfying.

C-Style error handling is basicaly "returning an error code when the application
failed". Here is short example.

``` cpp
int find_slash(const char *str)
{
    int i = 0;

    while (str[i] && str[i] != '/')
          i++;

    if (str[i] == '\0')
        return -1; //Error code

    //True value
    return i;
}

// . . .

if (find_slash(string) == -1)
{
        //error handling
}
```

### What's good here?

You can (and actualy, in C style, you do), right after the call, directly handle
the error by eventualy displaying a message and terminating the application,
or just retrieving the last state, aborting the computation...

When you wan't to find where the error handling is made, you just have to look at
the __function call__. It's __right__ after.

### What's bad here?

As some may have told you, it mix error handling with proper "execution flow".
When you read linearly the code as the code execute, you read sometime error
handling, and some time program logic. That's really bad, because you would like to
read either "program logic" or "error handling".

Also notice that you are limited to an "error code". If you want to provide
more informations, you'll have to create some functions (like errstr) and / or
use global variables;

## Do it C++-style: exceptions

When C was "enhanced" to c++, a new way of handling errors was introduced. It
was "exceptions". Exceptions are a way of breaking the normal flow of your code
by "throwing" an error that will be "catched" somewhere else. Again, a short example.

``` cpp
int find_slash(const char *str)
{
    int i = 0;

    while (str[i] && str[i] != '/')
          i++;

    if (str[i] == '\0')
        throw AnException("Error message");

    //True value
    return i;
}

// . . .

try
{
    find_slash(string);
}
catch(AnException& e)
{
   //Handle exception
}
```


### What's good here?

Program logic and error handling are separated. On one side you can see how the function
whork and when it fail, on the other part you can see what it does if it failed.
It's pretty, simplify reading of both error handling and program logic.

The second point is that now you can give as many information as you like, since
you can put what you want in your MyException object.

## What's bad here?

Writing exceptions well is verbose. You need an exception tree, if possible not too big, so that you can catch only the exception you are interesting it. Also, you need some
error codes inside, to know exactly what hapenned, retrieve an error message, etc.
Writing exceptions class __is__ verbose, it's the cost of the flexibility given by the
ability to handle more informations embeded as an error.

The way encouraged by this philosophy is to postpone error handling as late as possible,
so that users can have the highest flexibility to handle errors. Thats theoricaly a good
thing, but…

In big applications, you __can't__ find esily where the error is handled. When you wan't
to know what would be the path of an error, you have to jump across files, functions,
and it's really hard to find, if you introduce a new exception deep in the "call ~graph~ tree" (I mean the graph you would have if you draw each function call) then you have to
figure out where it should be handled, if it is already handled, in which places. It's
really hard when the application is big, old, not really well writen everywhere. Actualy
it's the case of most of the commercial projects. And that is that.

That's what make me state "Exceptions are dangerous". Of course, they provide a pretty
well to handle error… but only when you are working on small project, where the
"call graph" is simple and easy to get in the mind.

## The error box pattern

I call it a pattern, so that people don't be afraid. At the end, I'll call it with it's
right name, so please, be patient.

The main idea is creating a box that can contain either an error message or a value. We
will limit ourself to a string, and nothing more, because it's actualy not so easy to
implement. We will try to keep the syntax readable, understandable, for use cases.
We won't handle correctly copy constructor, more than one parameter functions,
and rvalues. We will keep it as simple as possible.

Let's start with an example of use:

``` cpp
E<int> find_slash(const char* str)
{
    int i = 0;

    while (str[i] && str[i] != '/')
          i++;

    if (str[i] == '\0')
        return fail<int>("Error message");

    //True value
    return ret(i);
}

// . . .

auto v = find_slash(string);
if(!v)
{
    //Handle exception
}
```

At the first look, it may look like it is the same thing as the C-style. But it isn't.
To see that, you have to look at more than one function call.

``` cpp
E<int> find_slash(const char*);
E<int> do_some_arithmetic(int);
E<std::string> format(int);
E<void> display(std::string);

auto v = ret(string)
         .bind(find_slash)
         .bind(do_some_arithmetic)
         .bind(format)
         .bind(display);

if(!v)
{
    //Handle error
}
```

Ok, so what happened ? Bind is a member function that takes your function, and tries
to apply it. If there is a value in the "error box", then it aplies to function, and then
returns an error box (the compiler won't let you give a function that doesn't return
an error box).

So, we chained the call to find_slash, do_some_arithmetic, format and display. None
of them can take an error box, but thanks to bind, we "transformed" the functions `E<something_out> f(something_in)` to
functions of type `E<something_out> f(E<something_in>)`.


### What's good here?

Once again, program logic (the chain) and error handling (the if) are separated. Like
with exception, we have the ability to read the chain and don't think about where
the execution is interrupted. Actualy, the chain __may__ be interrupted at __any__ step.
But we can think as if no error is happening and check quickly if our logic is right.

Of course, typing will prevent you from binding format after a display, so we
don't lose any typing capabilities.

Notice that we aren't calling any of those function in any other. We are "composing"
them at the end. That is the key to make it works. You should, write small modular
functions (hey, look at that: you __should__ write generic code so that it can work)
that accept a value, then compute an other "new" value, or fail. And at each
step, you don't have to think about where an exception may interrupt your control
flow and take care of maintain you stuff in a valid state (exception safety is basically
beeing always looking at each call you do, and figure out if the function can interrupt
your flow and what will happen if it does). For this point, it's "safer".

As with exceptions, we can have "as many information as you like", although in this
post we will write it "half template", so that it's a bit easier to understand.

We can easily locate __where__ error handling happens. It's always after a chaining
(unless the value is chained again). We have now a big execution flow, without
interruption, and small error handling flow, easy to locate. When a new error is added,
you just have to find the calls, following the chains, and you'll get back directly to
the handling, and add what's needed.
Big projects are more "linear" and easy to read.

### What's bad here?

First, it's new. It's not integrated to C++, it's not the standard way, and with the
stl you'll still have to use exceptions.

It's still a bit too much verbose for my taste. The need to write explicitely the type in
`fail<int>("...")` is bad. If you had a polymorphic error type, then it's worst because
you'll have to write `fail<return_type, error_type>("...")`.

It do not provide an easy way to call a function that need more than one argument.
In some other languages, you have "applicative" types and currification that solve it
nicely, but that's far from what you may expect from c++. I'm more likely to think that
a `bind2(E<a>, E<b>, f)` and `bind3(E<a>, E<b>, E<c>, f)` function could be used.
Variadic templates may help.

To get the value "out of the box", we have to check if the box is really a value,
and then call a "to_value" method. And we have no means from doing it without
checking. What we would like to have is "deconstruction" of objects, but this doesn't
exist in c++, and it's not the kind of things you just have to say "hey, let's
add it to the next standard".

At the moment, I don't know how you could adapt it to member functions. If you have
an idea, try it, and if it works, tell us :)


## Implementing a monadic error handling

I unleashed the demons, I wrote the name from dark magic. It's [monad][monad-haskell]. You can think of
monad as "box" containing a value in a "context". For example, a box that contains
either a value or nothing is a monad (I let you write the implementation as an exercise).
This concept was heavily used and abused by the language [Haskell][haskell-wikipedia],
so if you are curious you can have a look at my [french introduction crash course][haskell-intro].

A bit stranger, lists are monad. They are "a value" in a context, from a certain
point of view.

Let's start by implementing the E class you saw above. I'm relying on C++11 `decltype`
and `auto -> decltype` wich allow figuring out the type of things from expressions. It's
really useful.

The bind function is a bit strange, but it does what I stated previously.

``` cpp

/*
This is the "Either String" monad, as a way to handle errors.
*/

template
<typename T>
class E
{
private:
    //The value stored
    T m_value;
    //The error message stored
    std::string m_error;
    //A state. True it's a value, false it's the message.
    bool m_valid;

    E()
    {}

public:
    //Encapsulate the value
    static E ret(T v)
    {
        E box;
        box.m_value = v;
        box.m_valid = true;
        return box;
    }

    //Encapsulate an error
    static E fail(std::string str)
    {
        E box;
        box.m_error = str;
        box.m_valid = false;
        return box;
    }

    //True if it's a valid value
    operator bool() const
    {
        return m_valid;
    }

    //Deconstruct an E to a value
    T to_value() const
    {
        //It's a programmer error, it shouldn't happen.
        if (!*this)
        {
            std::cerr << "You can't deconstruct to a value from an error" << std::endl;
            std::terminate();
        }
        return m_value;
    }

    //Deconstruct an E to an error
    std::string to_error() const
    {
        //It's a programmer error, it shouldn't happen.
        {
            std::cerr << "You can't deconstruct to an error from a value" << std::endl;
            std::terminate();
        }
        return m_error;
    }

    friend std::ostream& operator<< (std::ostream& oss, const E<T>& box)
    {
        if (box)
            oss << box.m_value;
        else
            oss << box.m_error;
        return oss;
    }

    template<typename F>
    inline
    auto bind(F f) -> decltype(f(m_value))
    {
        using type = decltype(f(m_value));
        if (*this)
            return f(m_value);
        else
            return type::fail(m_error);
    }
};

```

I also overloaded the `<<` operator, so that it's easier to see what the box contains.
We don't really need it, and it may be a good idea to remove it for a "true" usage.

If you look at this, and the example, we need a "E<void>" type, but it won't work as if.
We need a special instance for void. It's exactly the same thing, except that a
value is now an "empty box".

``` cpp
/*
    Special instance for void
*/

template<>
class E<void>
{
private:
    std::string m_error;
    bool m_valid;

    E()
    {}

public:
    //Encapsulate the value
    static E ret()
    {
        E box;
        box.m_valid = true;
        return box;
    }

    //Encapsulate an error
    static E fail(std::string str)
    {
        E box;
        box.m_error = str;
        box.m_valid = false;
        return box;
    }

    //True if it's a valid value
    operator bool() const
    {
        return m_valid;
    }

    //Déconstruct an E to a value
    void to_value() const
    {
        //It's a programmer error, it shouldn't happen.
        if (!*this)
        {
            std::cerr << "You can't deconstruct to a value from an error" << std::endl;
            std::terminate();
        }
    }

    //Deconstruct an E to an error
    std::string to_error() const
    {
        //It's a programmer error, it shouldn't happen.
        if (*this)
        {
            std::cerr << "You can't deconstruct to an error from a value" << std::endl;
            std::terminate();
        }
        return m_error;
    }

    friend std::ostream& operator<< (std::ostream& oss, const E<void>& box)
    {
        if (box)
            oss << "()";
        else
            oss << box.m_error;
        return oss;
    }

    template<typename F>
    inline
    auto bind(F f) -> decltype(f())
    {
        using type = decltype(f());
        if (*this)
            return f();
        else
        return type::fail(m_error);
    }
};
```

Oh, we don't spoke about `ret` and `fail`. Actually, they are just wraper around
the `xxx::fail` and `xxx::ret` functions.

``` cpp
/*
   Then, I introduced those simple functions, to reduce the
   call to something readable/writable
 */
template <typename T>
inline
E<T> ret(T v)
{
    return E<T>::ret(v);
}

template <typename T>
inline
E<T> fail(std::string err)
{
    return E<T>::fail(err);
}
```

There, we are, you can compile and play with the example above.

If you want more stuff to play with, I also have this "more realistic" example:

``` cpp
/*
    Here comes a use case.
*/

// What a user would see:

//Return a value in an error context
template <typename T> inline
E<T> ret(T v);
//Fail in an error context of type T
template <typename T> inline
E<T> fail(std::string err);

// What a user would write:

typedef std::vector<std::string> vs;
typedef std::string str;

//Parse a +- formated string.
//If a letter is prefixed by +, then the function toupper is applied.
//''                                              tolower is applied.
//Non alphabetical (+ and - excepted) aren't alowed.
//Words are cut on each space ' '. Other blank characters aren't alowed.
E<vs> parse(std::string str)
{
    int mode = 0;
    vs vec;

    if (str.empty())
        return fail<vs>("Empty string aren't allowed");

    std::string stack;
    for(int i = 0; str[i] != '\0'; i++)
    {
        switch(str[i])
        {
        case '-':
            mode = 1;
            break;
        case '+':
            mode = 2;
            break;
        case ' ':
        {
            if(!stack.empty())
                vec.push_back(stack);
            stack.resize(0);
            mode = 0;
            break;
        }
        default:
        {
            if (!isalpha(str[i]))
                return fail<vs>("Only alpha characters are allowed");
            if (mode == 1)
                stack.push_back(tolower(str[i]));
            else if (mode == 2)
                stack.push_back(toupper(str[i]));
            else
                stack.push_back(str[i]);
            mode = 0;
            break;
        }
        }
    }
    if(!stack.empty())
        vec.push_back(stack);

    return ret(vec);
}

//Take the first word and append it to the begining of all other words.
//Vec should contain at least one element.
E<vs> prefixy(vs vec)
{
    if (vec.empty())
        return fail<vs>("Can't add prefixes on an empty table");

    std::string prefix = vec.front();
    vs out;

    for (auto s : vec)
    {
        if (prefix == s)
            continue;
        out.push_back(prefix + s + "^");
    }

    return ret(out);
}

//Concatenate all strings as a big string. Vec should contain data.
E<std::string> concat(vs vec)
{
    std::string output;

    if (vec.empty())
        return fail<str>("Empty vectors aren't allowed");

    for (auto s : vec)
        output += s;

    if (output.empty())
        return fail<str>("No data found");
    return ret(output);
}

int main()
{
    typedef std::string s;

    //Parse some string, show how error interrupt computation of the "chain".
    std::cout << ret((s)"+hello    -WORLD").bind(parse).bind(prefixy).bind(concat) << std::endl;
    std::cout << ret((s)"+hello Hello  Hello").bind(parse).bind(prefixy).bind(concat) << std::endl;
    std::cout << ret((s)"+   ").bind(parse).bind(prefixy).bind(concat) << std::endl;
    std::cout << ret((s)"+hi").bind(parse).bind(prefixy).bind(concat) << std::endl;

    //Play with lambda to "replace" a value if it's not an error.
    std::cout << ret((s)"Some string").bind([](const std::string&) {return fail<s>("Failed");});
    std::cout << ret(23).bind([](const int) {return ret(42);});
    std::cout << fail<int>("NaN").bind([](const int) {return ret(42);});

    return 0;
}
```

[monad-haskell]: /haskell-pure-3/
[haskell-wikipedia]: https://en.wikipedia.org/wiki/Haskell_(programming_language)
[haskell-intro]: /haskell-pure-1/