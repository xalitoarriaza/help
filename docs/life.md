---

template:      article
title:         Meaning of life
lead:          The universe, and everything.
naviTitle:     Life
noToc:         1

keywords:
    - 42
    - Hitchhiker's Guide to the Galaxy
    - Hitchhiker
    - Galaxy
    - Guide

tags:
    - advanced

externalLinks:
    - http://en.wikipedia.org/wiki/Phrases_from_The_Hitchhiker%27s_Guide_to_the_Galaxy

---

```php
echo (array_sum(array_map(function ($num) use (&$count) {
    return $num / $count;
}, array_filter(range(1, 99), function ($num) use (&$count) {
    return !($num % 3) && $count ++ > 10;
}))) >> 3) + $count / 528 * -- $count;
```

Of course: only an approximation.