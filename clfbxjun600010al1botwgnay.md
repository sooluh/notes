---
title: "Create My Own MessageFormat"
datePublished: Fri Mar 17 2023 02:37:42 GMT+0000 (Coordinated Universal Time)
cuid: clfbxjun600010al1botwgnay
slug: create-my-own-messageformat
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685070155871/810362a1-b8fa-47ef-a6ab-08c29caf5b2e.png
tags: javascript, regex

---

Some time ago, I was fascinated by the [intl-messageformat](https://formatjs.io/docs/intl-messageformat) package, and I always use it whenever a message format is needed.

The features are quite complete; I'm overwhelmed by it, but it turns out I only need the placeholder part; The other features are just the cherry on top, so I decided to make my placeholder.

The first thing that came to mind was [regex](https://en.wikipedia.org/wiki/Regular_expression), so I went straight to regex101 to do a little experimentation and test some of the strings normally used to create the placeholders.

```markdown
My name is {name}!
My name is { name }!
My name is {name }!
My name is { name}!
My name is {  name  }!
```

Those are some samples that may be formatted with dynamic variables in the future. Some don't use spaces at all, some use only one space in the prefix and the suffix, some use spaces only in the prefix or only in the suffix, or some may even have more than one space between them.

Because of the space condition, I used `\s+` to detect the space, and I added `?` to make it optional. So I optionally grouped the space tokens into `(\s+)?`.

So I complete the regular expression as below, where I detect the open and closed curly and add a backslash to escape them both. Then, following the sample, I added the keyword `name` to detect that the keyword exists.

```markdown
\{(\s+)?name(\s+)?\}
```

At least we have our regular expressions set up, so it's time to implement them into my current favorite language, TypeScript. Hmm, this is JavaScript, isn't it? Maybe I'll just apply it to JavaScript then.

```javascript
function parser(template, name) {
  return template.replace(/\{(\s+)?name(\s+)?\}/, name)
}

console.log(parser('My name is { name }!', 'John Doe'))
// My name is John Doe!
```

And of course, it only works for one placeholder; what if we have multiple keywords with the name `name`? Does it still work? Of course not, so I added the `gi` regex flag. `g` means globally, and `i` mean insensitive.

```javascript
function parser(template, name) {
  return template.replace(/\{(\s+)?name(\s+)?\}/gi, name)
}

console.log(parser('{ name } my name is { name }!', 'John'))
// John my name is John!
```

But maybe I need this for all the keywords with the same name; how do I solve it? Here, I use a for-loop to make things easier; if you know of a better way, please leave a comment.

```javascript
function parser(template, params) {
  for (key in params) {
    template = template.replace(/\{(\s+)?name(\s+)?\}/gi, params[key])
    // how do i make it dynamic? this is just for the name keyword, huh :(
  }
  return template
}
```

Do you see the problem? Yes, it has to be dynamic, but how do I handle it? I don't know how myself, at least before encountering the RegExp object class. I don't know if this is the best practice or not, but it would be helpful to receive some advice from you.

```javascript
function parser(template, params) {
  for (key in params) {
    let regex = new RegExp(`\{(\s+)${key}(\s+)\}`, 'gi')
    template = template.replace(regex, params[key])
  }
  return template
}
```

It's not working, is there something wrong? Maybe because I have too many sins? Astaghfirullahaladzim.

It turns out that objects of class RegExp don't work that way, we should add another backslash before the backslash. Hah? I mean, for each backslash, we have to give it another backslash, as shown below.

```javascript
function parser(template, params) {
  for (key in params) {
    let regex = new RegExp(`\\{(\\s+)${key}(\\s+)\\}`, 'gi')
    template = template.replace(regex, params[key])
  }
  return template
}
```

I don't understand why it has to be like that; I'm still looking for an explanation, but I couldn't find it (maybe I'm just not observant or even too lazy to search hehe).

Finally, before I close, maybe you want to try it. You can run the pen in my CodePen; [let's see](https://codepen.io/sooluh/pen/PodexKQ) (see console).

For now, that's all I can share with you, because I'm just an ordinary human being who has lots of flaws, and perfection belongs only to Allah. Hopefully, I shared something useful, once again, thank you.