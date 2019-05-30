---
title: Laravel mapSpread 例子
date: 2019-01-04 10:53:00
tags: [Laravel, PHP]
---

```php
$collection = collect(range(1, 9));
 
$chunks = $collection->chunk(2);
 
$labeld = $chunks->mapSpread(function ($odd, $even) {
    return "Odd: {$odd} Even: {$even}";
});
 
$numbers = collect([3, 5, 7]);
$words = collect(['three', 'five', 'seven']);
 
$results = $numbers->zip($words)
    ->mapSpread(function ($number, $word) {
        return $number . '=' . $word;
    });
 
dd($labeld->toArray(), $results->toArray());
```

![x](/images/11.png)
