---
layout: post
title:  "Pagination Buttons With Related Html Table"
date:   2017-02-27 10:08:07 +0100
categories: [html, css, angular2, bootstrap, functional programming]
---
I recently was looking for a solution to generate and manage pagination buttons related to a large collection inside a table with Angular2. I would use it in a modal page such that the table doesn’t override it. I first try to attach a scroll bar to the tbody of the table to limit the size. This solution used css tricks and was not consistent according to the navigators.

Finally, I start implementing my own Angular2 component. This component could be attached to any collections. My start developing incrementally and I was quickly face to poor quality code and mutable state to determine the context. I say stop and start isolate the main functionality. It provide a list of buttons :

* Given a collection and the current page
* Provide the list of Strings showing maximum 10 elements and comprising page number or ‘..’ if the number of pages is greater than 10.

This illustrates the objective of the paginated buttons-group: 
![Pagination buttons]({{ site.url }}/assets/pagin1.jpg)

Considering I like functional programming even if I mainly use imperative paradigm, I try I first version in Haskell before reimplementing in JS.

Haskell Version:
{% highlight haskell %}
paginate :: Int -> Int -> [String]
paginate page activep
    | page == 0                 = []
    | page <= 10                = map show [1..page]
    | page > 10 && activep <= 5 = map show [1..7] ++ [".."] ++ map show [page-1, page]
    | activep >= page -5        = map show [1..2] ++ [".."] ++ map show [page-6..page]
    | otherwise                 = map show [1..2] ++ [".."] ++ map show [activep-1..activep+2] ++ [".."] ++ map show [page-1, page]
{% endhighlight %}

Then, I think "great". Let keep this version and look at how integrating it with a TypeScript-Angular2 project. I looked for some ways to compile Haskell to JS with ghcjs but for now, the project is not very robust. Other solutions: PureScript or ELM. I chose Purescript instead of ELM because this last focus on view.

I change the syntax a bit and arrive to this version: 
{% highlight haskell %}

mapToString :: forall a. ( Show a ) => Array a -> Array String
mapToString = map show

paginate :: Int -> Int -> Array String
paginate page activep
    | page == 0                 = []
    | page <= 10                = mapToString $ 1..page
    | page > 10 && activep <= 5 = (mapToString $ 1..7) <> [".."] <> (mapToString [page-1, page])
    | activep >= page -5        = (mapToString $ 1..2) <> [".."] <> (mapToString $ (page-6)..page)
    | otherwise                 = (mapToString $ 1..2)
                                    <> [".."]
                                    <> (mapToString $ (activep-1)..(activep+2))
                                    <> [".."]
                                    <> (mapToString [page-1, page])
{% endhighlight %}

Nice. It works perfectly with my Angular2 project but if I look at the generated JS compressed code: ~32000 characters and 113 Kio. It includes lots of PuresScript standard libraries.

Ok, I suggest using PureScript in projects including lots of PureScript code. But just for a couple of functionality, it's not wise. So I confess the defeat and write a TypeScript + Lodash version. Finally, this is not so bad: 

{% highlight haskell %}
private paginate(activePage, size) : string[] {
  size = Math.ceil(size);
  if (size == 0){
    return [];
  } else if (size <= 10) {
    return _.range(1, size+1).map( i => i.toString() );
  } else if (size > 10 && activePage <= 5) {
    return _.range(1, 8)
      .map( i => i.toString() )
      .concat("..")
      .concat( _.range(size -1, size+1).map( i => i.toString() ));
  } else if (activePage >= size -5) {
    return _.range(1, 3)
      .map( i => i.toString() )
      .concat("..")
      .concat( _.range(size -6, size+1).map( i => i.toString() ));
  } else {
    return _.range(1, 3)
      .map( i => i.toString() )
      .concat("..")
      .concat( _.range(activePage-1, activePage+3).map( i => i.toString() ))
      .concat("..")
      .concat( _.range(size-1, size+1) );
  }
}
{% endhighlight %}

You observed that a transform number so string with the map method. Js is weakly typed, so it is not mandatory but it is consistent for me.

<iframe style="width: 100%; height: 600px" src="https://embed.plnkr.co/c1oNAW/" frameborder="0" allowfullscren="allowfullscren"></iframe>
Check out the final Angular2 version on [Plunker][plunker-link] and the [Gist][gist-link] repository for Haskell, PureScript and TypeScript version of the function paginate.


[plunker-link]: https://embed.plnkr.co/ggTVQD2NhtfhpJRNPT80/
[gist-link]: https://gist.github.com/jcavat/64338156116a5aaff93e892d187801df

