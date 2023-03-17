---
title: "Create My Own MessageFormat"
datePublished: Fri Mar 17 2023 02:37:42 GMT+0000 (Coordinated Universal Time)
cuid: clfbxjun600010al1botwgnay
slug: messageformat
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678936909799/c2f5dd51-8967-4bd9-8c2f-ff2840b8ef72.png
tags: javascript, regex

---

Some time ago I was fascinated by the [intl-messageformat](https://formatjs.io/docs/intl-messageformat) package and I always use it wherever message format is needed.

The options are quite complete, I'm even torn to use it, but on second thought, it turns out I don't need all the features, I just need the placeholder part, so I decided to make my own.

The first thing that came to my head was [regex](https://en.wikipedia.org/wiki/Regular_expression), so I went straight to regex101 to do a little experimentation and test some of the strings normally used to create placeholders.

```markdown
My name is {name}!
My name is { name }!
My name is {name }!
My name is { name}!
My name is {  name  }!
```

Those are some samples that may be in the future will be formatted with dynamic variables. Some don't use spaces at all, use one space in the prefix and the suffix, use spaces only in the prefix or only in the suffix, or there is more than one space between the two.

Because of the space condition, I used `\s+` to detect the space, and I added `?` to make it optional. So I optionally grouped the space tokens into `(\s+)?`.

So I complete the regular expression as below, where I detect the open and close curly and add a backslash to escape the curly both. Then following the sample, I added the keyword `name` to detect that the keyword exists.

```markdown
\{(\s+)?name(\s+)?\}
```

At least we have our regular expression set up, so it's time to implement it into my current favorite language, TypeScript. Hmm, this is JavaScript, isn't it? maybe I'll just apply it to JavaScript then.

```javascript
function parser(template, name) {
  return template.replace(/\{(\s+)?name(\s+)?\}/, name)
}

console.log(parser('My name is { name }!', 'John Doe'))
// My name is John Doe!
```

And of course, it only works for one placeholder, what if the "name" keyword is multiple? does it still work? of course not, so I added the `gi` regex flag. `g` means globally, and `i` mean insensitive.

```javascript
function parser(template, name) {
  return template.replace(/\{(\s+)?name(\s+)?\}/gi, name)
}

console.log(parser('{ name } my name is { name }!', 'John'))
// John my name is John!
```

But maybe I need this for multiple keywords at once, how do I solve it? here I use a for-loop to make it easier, if any of you have an easier way, please leave a comment.

```javascript
function parser(template, params) {
  for (key in params) {
    template = template.replace(/\{(\s+)?name(\s+)?\}/gi, params[key])
    // how do i make it dynamic? this is just for the name keyword, huh :(
  }
  return template
}
```

Do you see the problem? yes, it has to be dynamic, how do I handle it, I don't know how myself, at least before encountering the [RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp) object class. I don't know if this is the best practice or not, but I need some advice from you 😁

```javascript
function parser(template, params) {
  for (key in params) {
    let regex = new RegExp(`\{(\s+)${key}(\s+)\}`, 'gi')
    template = template.replace(regex, params[key])
  }
  return template
}
```

Something's wrong, it's not working, is there something wrong? maybe because of my too many sins? Astaghfirullahaladzim.

It turns out that because objects of class RegExp don't work that way, we should add another backslash before the backslash. Hah? I mean, on each backslash, we need to duplicate it, like the following.

```javascript
function parser(template, params) {
  for (key in params) {
    let regex = new RegExp(`\\{(\\s+)${key}(\\s+)\\}`, 'gi')
    template = template.replace(regex, params[key])
  }
  return template
}
```

I don't understand why it has to be like that, I'm still looking for an explanation, but I can't find it (maybe I'm just not observant or even lazy to search haha).

Finally, before I close, maybe you want to try it, you can run the pen in my codepen, [let's see](https://codepen.io/sooluh/pen/PodexKQ) (see consoles).

Maybe that's all I can share, nothing more, because I'm just an ordinary human being who has lots of flaws, and perfection belongs only to Allah. Hopefully something useful, once again that's all and thanks.