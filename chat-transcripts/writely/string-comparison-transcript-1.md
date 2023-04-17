# Session With Explicit Guidance

In this chat session, I asked GPT-4 to implement an algorithm for compactly describing the difference between two
strings, subject to the constraints we faced when implementing Writely (the original version of Google Docs). I
provided explicit hints and guidance, somewhat as if it were a junior engineer who was stretching themselves.

To kick things off, I gave a carefully constructed prompt that was designed to give it the best possible chance of coming
up with a usable solution:

> *[Me] You are an experienced and talented software engineer, with a thorough knowledge of published algorithms, as well as a knack for finding out-of-the-box solutions to problems that have not been solved previously. Please write a JavaScript program to compare two strings, find the difference between them, and produce a compact representation of that difference. The representation must contain enough information to allow the second string to be reconstructed from the first string.This code needs to be able to run on old browsers such as IE 6, which had very inefficient JavaScript engines, so the code can't do much work; it must not iterate through each character individually. It is not necessary that the representation be optimally minimal, we can use an approach that creates a larger-than-optimal representation of the difference if this allows for fewer execution steps in the code. Also, don't rely on any libraries, you can only use the basic functions built into JavaScript.*

I’ve pointed out that it should prioritize execution speed, even if that means using more data than necessary. In practice, that was a judgement I needed to arrive at myself. It would be hard to evaluate GPT-4’s performance on such a judgement call, because it relies on a lot of context that GPT-4 doesn’t have; this is a topic I’ll return to in my next post. I also gave the explicit guidance that “it must not iterate through each character individually”, which is an important hint as to what might need to be done in order to achieve the necessary performance.

Here is GPT-4's first attempt:

> Given the constraints of not iterating through each character individually and avoiding libraries, one approach to solving this problem is to use the split() function, which is supported even by older browsers like IE 6. We can divide the strings into chunks and compare them, generating a compact representation of the difference. This method may result in a larger-than-optimal representation, but it allows for fewer execution steps.
> 
> Here's a simple JavaScript program that does this:

```
function createCompactRepresentation(str1, str2) {
    var chunkSize = 5;
    var diff = [];
    var chunks1 = Math.ceil(str1.length / chunkSize);
    var chunks2 = Math.ceil(str2.length / chunkSize);

    for (var i = 0; i < Math.max(chunks1, chunks2); i++) {
        var chunk1 = str1.slice(i * chunkSize, (i + 1) * chunkSize);
        var chunk2 = str2.slice(i * chunkSize, (i + 1) * chunkSize);

        if (chunk1 !== chunk2) {
            diff.push({index: i, chunk: chunk2});
        }
    }

    return diff;
}

function reconstructString(str1, diff) {
    var chunkSize = 5;
    var str2 = str1;

    for (var i = 0; i < diff.length; i++) {
        var start = diff[i].index * chunkSize;
        var end = start + diff[i].chunk.length;
        str2 = str2.slice(0, start) + diff[i].chunk + str2.slice(end);
    }

    return str2;
}

var str1 = "Hello, this is a test string.";
var str2 = "Hello, this is a modified test string.";

var diff = createCompactRepresentation(str1, str2);
console.log("Compact Representation:", diff);

var reconstructedStr2 = reconstructString(str1, diff);
console.log("Reconstructed String:", reconstructedStr2);
```

> This code defines two functions, `createCompactRepresentation` and `reconstructString`. The first one takes two strings as input, divides them into chunks, and compares the chunks to produce a compact representation of the difference. The second function takes the original string and the difference representation, and reconstructs the modified string.
> 
> This approach should work on older browsers like IE 6, and it doesn't rely on any external libraries. However, note that this solution is not optimally minimal, and its performance will depend on the chunk size chosen.

This isn’t a bad start! However, it suffers from two weaknesses. First, it compares the same number of characters each time. This isn’t ideal: if you use a small number like 5, it will take too many steps to get through the whole document; if you use a large number like 1000, you’ll upload an unnecessarily large amount of data. I pointed out the second weakness in my next chat prompt:

> *This is promising, but if there is an extra character inserted near the beginning of the string, no chunks will match, and so the difference representation will be very large. Can you come up with an approach that will produce a reasonably compact difference even if there is a small change anywhere in the string, including near the beginning?*

It responded to this by trying a standard algorithm. However, I cut it off, because that algorithm requires too many processing steps:

> Understood. In that case, we can use a more efficient approach based on the Longest Common Subsequence (LCS) algorithm, with some modifications. The LCS algorithm can identify the longest common subsequence of characters between two strings, which can be used to generate a reasonably compact representation of the difference. Here's a JavaScript program that uses a modified version of the LCS algorithm:

```
function lcsMatrix(str1, str2) {
    var matrix = [];

    for (var i = 0; i <= str1.length; i++) {
        matrix[i] = [0];
    }

    for (var j = 0; j <= str2.length; j++) {
        matrix[0][j] = 0   [I halted the output here]
```

> *I halted this response, because the approach is flawed. The first loop in the lcsMatrix function contains a loop whose iteration count is equal to the length of the first input string. As described in the original problem statement, iterating once per character is too many steps for the IE 6 JavaScript engine.*

GPT-4 then tried a different standard algorithm, which was no better:

> Apologies for the oversight. Given the constraint of not iterating through each character individually, we can try another approach using a rolling hash function. This method will allow us to compare chunks of the strings without iterating through each character while still providing a reasonably compact difference representation.
> 
> Here's a JavaScript program that uses a rolling hash function:

```
function hash(str, base, modulus) {
    var result = 0;
    for (var i = 0; i < str.length; i++) {
        result = (result * base + str.charCodeAt(i)) % modulus;
    }
    return result;
}

function createCompactRepresentation(str1, str2) {
    var chunkSize = 5;
    var base = 128;
    var modulus = 104729;
    var hash1 = hash(str1.slice(0, chunkSize), base, modulus);
    var hash2 = hash(str2.slice(0, chunkSize), base, modulus);
    var power = Math.pow(base, chunkSize - 1) % modulus;
    var diff = [];

    for (var i = 1; i < str1.length || i < str2.length; i++) {
        if (hash1 !== hash2) {
            diff.push({index: i - 1, char: str2.charAt(i - 1)});
        }

        if (i < str1.length) {
            hash1 = (hash1 - str1.charCodeAt(i - 1) * power) * base + str1.charCodeAt(i) % modulus;
        }

        if (i < str2.length) {
            hash2 = (hash2 - str2.charCodeAt(i - 1) * power) * base +   [I halted the output here]
```

> *I halted this response as well. The "hash" function iterates through characters one a a time, so this algorithm is still too expensive.*

GPT-4’s next tried something different:

> I apologize for the oversight again. Since we cannot iterate through each character individually, we can use a simple approach that finds the first differing character and the last differing character between the two strings. This approach may not always produce the most compact difference representation, but it will provide a reasonably compact representation in many cases without iterating through each character.
> 
> Here's a JavaScript program that implements this approach:

```
function findFirstDifferingIndex(str1, str2) {
    var minLength = Math.min(str1.length, str2.length);
    for (var i = 0; i < minLength; i += 5) {
        if (str1.charAt(i) !== str2.charAt(i)) {
            return i;
        }
    }
    return minLength;
}

function findLastDifferingIndex(str1, str2) {
    var minLength = Math.min(str1.length, str2.length);
    for (var i = 1; i <= minLength; i += 5) {
        if (str1.charAt(str1.length - i) !== str2.charAt(str2.length - i)) {
            return str2.length - i;
        }
    }
    return minLength;
}

function createCompactRepresentation(str1, str2) {
    var firstDiffIndex = findFirstDifferingIndex(str1, str2);
    var lastDiffIndex = findLastDifferingIndex(str1, str2);
    var diff = {
        start: firstDiffIndex,
        end: lastDiffIndex,
        content: str2.slice(firstDiffIndex, lastDiffIndex + 1)
    };

    return diff;
}

function reconstructString(str1, diff) {
    return str1.slice(0   [I halted the output here]
```

This code steps through the document from both ends, checking every 5th character. It incorporates one of the key ideas, namely, working from both ends of the document to converge on the changed region. However, it’s still too slow (checking every 5th character is a lot); more importantly, it’s badly flawed, as I explain in my next prompt:

> *Finding the first and last differing characters seems like a promising idea. However, this implementation is flawed. For instance, findFirstDifferingIndex only looks at every 5th character, so it will not notice if the 2nd characters of the strings are different.*

> You're right, my previous implementation was flawed. Let me try a new approach that still finds the first and last differing characters but with fewer iterations. We can use a binary search-like method to find the first and last differing characters. The code will have a logarithmic number of iterations with respect to the string length.
> 
> Here's a JavaScript program that implements this approach:

```
function binarySearchDifferingIndex(str1, str2, start, end, findLast) {
    if (start > end) {
        return findLast ? end + 1 : start;
    }

    var mid = Math.floor((start + end) / 2);
    if ((findLast && str1.charAt(mid) === str2.charAt(mid)) || (!findLast && str1.charAt(mid) !== str2.charAt(mid))) {
        return binarySearchDifferingIndex(str1, str2, start, mid - 1, findLast);
    } else {
        return binarySearchDifferingIndex(str1, str2, mid + 1, end, findLast);
    }
}

function createCompactRepresentation(str1, str2) {
    var firstDiffIndex = binarySearchDifferingIndex(str1, str2, 0, Math.min(str1.length, str2.length) - 1, false);
    var lastDiffIndex = binarySearchDifferingIndex(str1, str2, firstDiffIndex, Math.min(str1.length, str2.length) - 1, true);

    var diff = {
        start: firstDiffIndex,
        end: lastDiffIndex,
        content: str2.slice(firstDiffIndex, lastDiffIndex + 1)
    };

    return diff;
}

function reconstructString(str1, diff) {
    return str1.slice(0, diff.start) + diff.content + str1.slice(diff.end + 1);
}

var str1 = "Hello, this is a test string.";
var str2 = "Hello, this is a modified test string.";

var diff = createCompactRepresentation(str1, str2);
console.log("Compact Representation:", diff);

var reconstructedStr2 = reconstructString(str1, diff);
console.log("Reconstructed String:", reconstructedStr2);
```

> This code defines a function `binarySearchDifferingIndex` that finds the first or last differing character index between two strings using a binary search-like approach. The `createCompactRepresentation` function uses this to find the first and last differing characters and generate a compact representation of the difference. The `reconstructString` function takes the original string and the difference representation and reconstructs the modified string. This approach should work for the constraints given, including old browsers like IE 6.

GPT-4 has now found another key insight: it is using a binary search to narrow down the modified region of the document. However, it still has the same flaw, as I explain again:

> *The binary search seems like a nice idea, but it is still just spot-checking individual characters, and so it may fail to notice modified sections of the input strings if the changes fall in between the spot checks. For instance, when given the inputs "nice big dog" and "mean big dog", it fails to notice any changes, and outputs "nice big dog" as the reconstructed string.*

GPT-4’s response was to go back to the idea of checking 5 characters at a time:

> I apologize for the confusion. You are correct that the binary search approach might miss modified sections between spot checks. Given the constraints, it seems challenging to provide a solution that guarantees accurate results while avoiding individual character iteration. However, I can propose a simple approach that checks for differing characters with a fixed step size. Although this approach may not produce the most compact difference representation, it will be more likely to identify changes than the binary search approach.
> 
> Here's a JavaScript program that implements this approach:

```
function createCompactRepresentation(str1, str2) {
    var stepSize = 5;
    var diff = [];

    for (var i = 0; i < str1.length || i < str2.length; i += stepSize) {
        var start = i;
        var end = Math.min(i + stepSize, Math.min(str1.length, str2.length));

        if (str1.slice(start, end) !== str2.slice(start, end)) {
            diff.push({index: start, content: str2.slice(start, end)});
        }
    }

    return diff;
}

function reconstructString(str1, diff) {
    var str2 = str1;
    var offset = 0;

    for (var i = 0; i < diff.length; i++) {
        var start = diff[i].index + offset;
        var end = start + diff[i].content.length;
        str2 = str2.slice(0, start) + diff[i].content + str2.slice(end);
        offset += diff[i].content.length - (end - start);
    }

    return str2;
}

var str1 = "nice big dog";
var str2 = "mean big dog";

var diff = createCompactRepresentation(str1, str2);
console.log("Compact Representation:", diff);

var reconstructedStr2 = reconstructString(str1, diff);
console.log("Reconstructed String:", reconstructedStr2);
```

> This code uses a fixed step size to check for differences between the two strings. The `createCompactRepresentation` function generates a compact representation of the difference, and the `reconstructString` function reconstructs the modified string from the original string and the difference representation. This approach should work for the constraints given, including old browsers like IE 6, but it might not capture all changes in certain cases, as it doesn't iterate through each character individually.

At this point, I broke off the session. GPT-4 successfully happened upon all of the important elements for a solution: working from both ends, comparing multiple characters at once, and using a binary search to efficiently mange its steps. However, it showed no sign of being able to combine these elements into a single solution; instead, it seemed to be flailing around somewhat randomly.
