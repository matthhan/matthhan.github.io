---
layout: post
title:  "Vimscript's <i>substitute</i> Function Uses Strange Patterns"
date:   2018-06-28 18:30:00 +0200
categories: vimscript vim
---
So recently I was working on my .vimrc file. Specifically, I wanted an easy way
to toggle the case of an identifier between __camelCase__ and __snake_case__.  
Normally, converting to __snake_case__ could be achieved with a substitute
__command__ by marking the word that you want to change and then doing
`:s/\u/_\l&/g`, but obviously this is too much to type every time.  Also, we
want to be able to change the variable name everywhere in the file. Luckily,
vimscript exposes a `substitute` __function__ with a signature that looks just
like the command: `substitute(expr,pattern,substitution,flags)`. So its
basically the same but you don't need the identifier, right?
I rejoiced and tried the following (with escaped backslashes due to being in vimscript):

{% highlight vimscript %} 
:function! ToSnakeCase(w) : 
  return substitute(a:w,"\\l","_\\u&","g") 
:endfunction 
{% endhighlight %}

But this does not seem to do the right thing. But why? I have no idea! However,
by an educated guess, I have found the right pattern that replaces lowercase
letters by uppercase.

Behold:

{% highlight vimscript %} 
:  substitute(a:w,"\\L","_\\L&","g") 
{% endhighlight %}

I.e. use an L instead of a U. I still don't have any idea *why* this is the
case but it seems to be working. Note how the character L has to be
capitalized. Lowercase does __not__ work.  In case anybody's interested, here
is the full code for the SwapCase functionality. Short cautionary Remark: this swaps the case
of the variable __file-wide__ and is not a semantic rename, so might cause problems
in big codebases and with public variables. Use with caution.

{% highlight vimscript %} 
:function! IsSnakeCase(w)
: return a:w =~ "_"
:endfunction
:function! ToCamelCase(w)
:  let w = a:w
:  let firstCharLower = substitute(w,w[0],"\\L&","")
:  let cased = substitute(firstCharLower,"_\\(.\\)","\\U\\1","g")
:  return cased
:endfunction
:function! SwapCase(w)
:  let w = a:w
:  if IsSnakeCase(w)
:    return ToCamelCase(w)
:  else
:    return ToSnakeCase(w)
:  endif
:endfunction
:function! SwapCaseGlobally()
:  let pos = getpos(".")
:  let w = expand("<cword>")
:  execute ("normal! :%s/" . w . "/" . SwapCase(w) . "/g\<enter>" )
:  call setpos('.',pos)
:endfunction
{% endhighlight %}

And then you can just bind `SwapCaseGlobally()` to the keybinding of your choice.
