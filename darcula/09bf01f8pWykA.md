---
layout: default
---

# Objective

The goal is to automate JavaScript deobfuscation. However, in order to do so, it's important to first understand how deobfuscation works in JavaScript to then look and see how to automate the process. 

# Finding a Place to Start

First things first, we need to examine the components of the obfuscated file. We see that the file is broken up into the following sections: 

1. A self-executing anonymous function
2. A variable definition
3. Two function definitions

Now, the order of execution dictates that the self-executing anonymous function will run first with the variable definition running second. Therefore, we must consider the follow point when automating JavaScript deobfuscation: 

> The order of operations is paramount

Now, let's take a closer look at the self-executing anonymous function: 

```js
(function (u, A) {
    const h = a0A,
        E = u();
    while (!![]) {
        try {
            const D =
                (parseInt(h(0x154)) / 0x1) * (-parseInt(h(0x152)) / 0x2) +
                (-parseInt(h(0x156)) / 0x3) * (parseInt(h(0x162)) / 0x4) +
                -parseInt(h(0x15b)) / 0x5 +
                -parseInt(h(0x151)) / 0x6 +
                parseInt(h(0x15e)) / 0x7 +
                (parseInt(h(0x159)) / 0x8) * (parseInt(h(0x157)) / 0x9) +
                (parseInt(h(0x15f)) / 0xa) * (parseInt(h(0x160)) / 0xb);
            if (D === A) break;
            else E["push"](E["shift"]());
        } catch (v) {
            E["push"](E["shift"]());
        }
    }
})(a0u, 0x6f0ff);
```

Let's take a moment and simplify certain aspects before we continue: 

1. `!![]` always evaluates to `true`
2. `u = a0u` and `A = 0x6f0ff`

```js
(function (u, A) {
    const h = a0A,
        E = a0u();
    while (true) {
        try {
            const D =
                (parseInt(h(0x154)) / 0x1) * (-parseInt(h(0x152)) / 0x2) +
                (-parseInt(h(0x156)) / 0x3) * (parseInt(h(0x162)) / 0x4) +
                -parseInt(h(0x15b)) / 0x5 +
                -parseInt(h(0x151)) / 0x6 +
                parseInt(h(0x15e)) / 0x7 +
                (parseInt(h(0x159)) / 0x8) * (parseInt(h(0x157)) / 0x9) +
                (parseInt(h(0x15f)) / 0xa) * (parseInt(h(0x160)) / 0xb);
            if (D === 0x6f0ff) break;
            else E["push"](E["shift"]());
        } catch (v) {
            E["push"](E["shift"]());
        }
    }
})(a0u, 0x6f0ff);
```

Now, to simplify things further, let's remove the outer layer of the self-execution anonymous function as we've already established that it executes first: 

```js
const h = a0A,
    E = a0u();
while (true) {
    try {
        const D =
            (parseInt(h(0x154)) / 0x1) * (-parseInt(h(0x152)) / 0x2) +
            (-parseInt(h(0x156)) / 0x3) * (parseInt(h(0x162)) / 0x4) +
            -parseInt(h(0x15b)) / 0x5 +
            -parseInt(h(0x151)) / 0x6 +
            parseInt(h(0x15e)) / 0x7 +
            (parseInt(h(0x159)) / 0x8) * (parseInt(h(0x157)) / 0x9) +
            (parseInt(h(0x15f)) / 0xa) * (parseInt(h(0x160)) / 0xb);
        if (D === 0x6f0ff) break;
        else E["push"](E["shift"]());
    } catch (v) {
        E["push"](E["shift"]());
    }
}
```

We can then substitute `h(...)` with `a0A(...)`: 

```js
const E = a0u();
while (true) {
    try {
        const D =
            (parseInt(a0A(0x154)) / 0x1) * (-parseInt(a0A(0x152)) / 0x2) +
            (-parseInt(a0A(0x156)) / 0x3) * (parseInt(a0A(0x162)) / 0x4) +
            -parseInt(a0A(0x15b)) / 0x5 +
            -parseInt(a0A(0x151)) / 0x6 +
            parseInt(a0A(0x15e)) / 0x7 +
            (parseInt(a0A(0x159)) / 0x8) * (parseInt(a0A(0x157)) / 0x9) +
            (parseInt(a0A(0x15f)) / 0xa) * (parseInt(a0A(0x160)) / 0xb);
        if (D === 0x6f0ff) break;
        else E["push"](E["shift"]());
    } catch (v) {
        E["push"](E["shift"]());
    }
}
```

We now need to find the value of `a0u()`: 

```js
function a0u() {
    const O = [
        "763343ZEEmqI",
        "10MjwbHE",
        "9850357GcgXRv",
        "createElement",
        "143668KLzuHC",
        "2744166fvKFHm",
        "159958nePvPH",
        "click",
        "1UiidKZ",
        "nofollow\x20noreferrer\x20noopener",
        "51QZYsCO",
        "9rBvnZg",
        "href",
        "6312632cMablh",
        "body",
        "953875DutVaJ",
        "rel",
        "remove",
    ];
    a0u = function () {
        return O;
    };
    return a0u();
}
```

We can see that this simply returns a array of strings. Let's plug this into our original work: 

```js
const E = [ "763343ZEEmqI", "10MjwbHE", "9850357GcgXRv", "createElement", "143668KLzuHC", "2744166fvKFHm", "159958nePvPH", "click", "1UiidKZ", "nofollow\x20noreferrer\x20noopener", "51QZYsCO", "9rBvnZg", "href", "6312632cMablh", "body", "953875DutVaJ", "rel", "remove", ];

while (true) {
    try {
        const D =
            (parseInt(a0A(0x154)) / 0x1) * (-parseInt(a0A(0x152)) / 0x2) +
            (-parseInt(a0A(0x156)) / 0x3) * (parseInt(a0A(0x162)) / 0x4) +
            -parseInt(a0A(0x15b)) / 0x5 +
            -parseInt(a0A(0x151)) / 0x6 +
            parseInt(a0A(0x15e)) / 0x7 +
            (parseInt(a0A(0x159)) / 0x8) * (parseInt(a0A(0x157)) / 0x9) +
            (parseInt(a0A(0x15f)) / 0xa) * (parseInt(a0A(0x160)) / 0xb);
        if (D === 0x6f0ff) break;
        else E["push"](E["shift"]());
    } catch (v) {
        E["push"](E["shift"]());
    }
}
```

Now, let's examine the `a0A()` function and integrate its logic into our work: 

```js
function a0A(u, A) {
    const E = a0u();
    return (
        (a0A = function (r, D) {
            r = r - 0x151;
            let v = E[r];
            return v;
        }),
        a0A(u, A)
    );
}
```

First, let's simplify it further and remove the `D` as it is never called as well as simplify how the function is presented: 

```js
function a0A(u, A) {
    const E = [ "763343ZEEmqI", "10MjwbHE", "9850357GcgXRv", "createElement", "143668KLzuHC", "2744166fvKFHm", "159958nePvPH", "click", "1UiidKZ", "nofollow\x20noreferrer\x20noopener", "51QZYsCO", "9rBvnZg", "href", "6312632cMablh", "body", "953875DutVaJ", "rel", "remove", ];

    r = r - 0x151;
    let v = E[r];
    return v;
}
```

Now, let's take a snapshot and examine how `a0A` is called and resolve these calls ourselves: 

```js
const D =
            (parseInt(a0A(0x154)) / 0x1) * (-parseInt(a0A(0x152)) / 0x2) +
            (-parseInt(a0A(0x156)) / 0x3) * (parseInt(a0A(0x162)) / 0x4) +
            -parseInt(a0A(0x15b)) / 0x5 +
            -parseInt(a0A(0x151)) / 0x6 +
            parseInt(a0A(0x15e)) / 0x7 +
            (parseInt(a0A(0x159)) / 0x8) * (parseInt(a0A(0x157)) / 0x9) +
            (parseInt(a0A(0x15f)) / 0xa) * (parseInt(a0A(0x160)) / 0xb);
```

Let's look at the first value: `a0A(0x154)` and solve it: 

```js
const E = [ "763343ZEEmqI", "10MjwbHE", "9850357GcgXRv", "createElement", "143668KLzuHC", "2744166fvKFHm", "159958nePvPH", "click", "1UiidKZ", "nofollow noreferrer noopener", "51QZYsCO", "9rBvnZg", "href", "6312632cMablh", "body", "953875DutVaJ", "rel", "remove", ];

r = 0x154 - 0x151; // 0x03
let v = E[0x03]; // createElement
return v;
```

Here, we see that the resulting value of `D` is `NaN`. However, let's inject a few `console.log` statements to see where it goes: 

```js
function a0u() {
    const O = [
        "763343ZEEmqI",
        "10MjwbHE",
        "9850357GcgXRv",
        "createElement",
        "143668KLzuHC",
        "2744166fvKFHm",
        "159958nePvPH",
        "click",
        "1UiidKZ",
        "nofollow\x20noreferrer\x20noopener",
        "51QZYsCO",
        "9rBvnZg",
        "href",
        "6312632cMablh",
        "body",
        "953875DutVaJ",
        "rel",
        "remove",
    ];
    a0u = function () {
        return O;
    };
    return a0u();
}

function a0A(u, A) {
    const E = a0u();
    return (
        (a0A = function (r, D) {
            r = r - 0x151;
            let v = E[r];
            return v;
        }),
        a0A(u, A)
    );
}

(function (u, A) {
    const h = a0A,
        E = u();
    while (!![]) {
        try {
            const D =
                (parseInt(h(0x154)) / 0x1) * (-parseInt(h(0x152)) / 0x2) +
                (-parseInt(h(0x156)) / 0x3) * (parseInt(h(0x162)) / 0x4) +
                -parseInt(h(0x15b)) / 0x5 +
                -parseInt(h(0x151)) / 0x6 +
                parseInt(h(0x15e)) / 0x7 +
                (parseInt(h(0x159)) / 0x8) * (parseInt(h(0x157)) / 0x9) +
                (parseInt(h(0x15f)) / 0xa) * (parseInt(h(0x160)) / 0xb);
            if (D === A) {
              console.log(E);
              break;
            }
            else E["push"](E["shift"]());
        } catch (v) {
            E["push"](E["shift"]());
        }
    }
})(a0u, 0x6f0ff);
```

Here, we get the following output: 

```js
[
  '2744166fvKFHm',
  '159958nePvPH',
  'click',
  '1UiidKZ',
  'nofollow noreferrer noopener',
  '51QZYsCO',
  '9rBvnZg',
  'href',
  '6312632cMablh',
  'body',
  '953875DutVaJ',
  'rel',
  'remove',
  '763343ZEEmqI',
  '10MjwbHE',
  '9850357GcgXRv',
  'createElement',
  '143668KLzuHC'
]
```

Now we have the correct order from which we can examine the exported function: 

```js
const r = (u) => {
    const T = a0A,
        A = document[T(0x161)]("a");
    (A[T(0x158)] = u), (A[T(0x15c)] = T(0x155)), document[T(0x15a)]["append"](A), A[T(0x153)](), setTimeout(() => A[T(0x15d)](), 0x1f4);
};
```

Let's simplify this further and resolve each entry of `T` to the corresponding value given our new string array: 

* `T(0x161)`: `0x161 - 0x151 = 0x10` which then is `createElement`
* `T(0x158)`: `0x158 - 0x151 = 0x07` which then is `href`
* `T(0x15c)`: `0x15c - 0x151 = 0x0b` which then is `rel`
* `T(0x155)`: `0x155 - 0x151 = 0x04` which then is `nofollow noreferrer noopener`
* `T(0x15a)`: `0x15a - 0x151 = 0x09` which then is `body`
* `T(0x153)`: `0x153 - 0x151 = 0x02` which then is `click`
* `T(0x15d)`: `0x15d - 0x151 = 0x0c` which then is `remove`

Now, let's simplify the exported function: 

```js
const r = (u) => {
    const A = document["createElement"]("a");
    (A["href"] = u), (A["rel"] = "nofollow noreferrer noopener"), document["body"]["append"](A), A["click"](), setTimeout(() => A["remove"](), 0x1f4);
};

export {r};
```

Interestingly enough, ChatGPT was able to deobfuscate the original sample into: 

```js
const r = (url) => {
    const anchor = document.createElement("a");
    anchor.href = url;
    anchor.rel = "nofollow noreferrer noopener";
    document.body.append(anchor);
    anchor.click();
    setTimeout(() => anchor.remove(), 500);
};

export { r };
```