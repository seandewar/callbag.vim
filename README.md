# callbag.vim

Lightweight observables and iterables for VimScript based on [Callbag Spec](https://github.com/callbag/callbag).

## Source Factories

| Implemented   | Name                                                   |
|---------------|--------------------------------------------------------|
| Yes           | create                                                 |
| Yes           | empty                                                  |
| Yes           | fromArray                                              |
| Yes           | fromEvent                                              |
| Yes           | interval                                               |
| Yes           | lazy                                                   |
| Yes           | never                                                  |
| Yes           | of                                                     |
| Yes           | throwError                                             |

## Sink Factories

| Implemented   | Name                                                   |
|---------------|--------------------------------------------------------|
| Yes           | forEach                                                |
| Yes           | subscribe                                              |

## Operators

| Implemented   | Name                                                   |
|---------------|--------------------------------------------------------|
| Yes           | combine                                                |
| Yes           | concat                                                 |
| Yes           | debounceTime                                           |
| Yes           | delay                                                  |
| Yes           | filter                                                 |
| Yes           | flatten                                                |
| Yes           | group                                                  |
| Yes           | map                                                    |
| Yes           | merge                                                  |
| Yes           | scan                                                   |
| Yes           | take                                                   |
| Yes           | takeUntil                                              |
| Yes           | takeWhile                                              |
| No            | concatWith                                             |
| No            | distinctUntilChanged                                   |
| No            | mergeWith                                              |
| No            | rescue                                                 |
| No            | retry                                                  |
| No            | skip                                                   |
| No            | throttle                                               |
| No            | timeout                                                |

## Utils

| Implemented   | Name                                                   |
|---------------|--------------------------------------------------------|
| Yes           | operate                                                |
| Yes           | pipe                                                   |

`pipe()`'s first argument should be a source factory.
`operate()` doesn't requires first function to be the source.

***Note** In order to support older version of vim without lambdas, callbag.vim explicitly doesn't use lambdas in the source code.*

## Difference with callbag spec

While the original callbag spec requires payload to be optional - `(type: number, payload?: any) => void`,
callbag.vim requires payload to be required. This is primarily due to limition on how vimscript functions works.
Having optional parameter and using `...` and `a:0` to read the extra args and then use `a:1` makes the code complicated.
You can use `callbag#undefined()` method to pass undefined.

## Example

```viml
let s:i = 0
function! s:log(x) abort
    let s:i += 1
    echom 'log ' . s:i . '   ' . a:x
endfunction

function! callbag#demo() abort
    call callbag#pipe(
        \ callbag#interval(1000),
        \ callbag#take(10),
        \ callbag#map({x-> x + 1}),
        \ callbag#filter({x-> x % 2 == 0}),
        \ callbag#map({x-> x * 1000}),
        \ callbag#forEach({x -> s:log(x) }),
        \ )
    call callbag#pipe(
        \ callbag#fromEvent(['TextChangedI', 'TextChangedP']),
        \ callbag#debounceTime(250),
        \ callbag#forEach({x -> s:log('text changed') }),
        \ )
    call callbag#pipe(
        \ callbag#fromEvent('InsertLeave'),
        \ callbag#forEach({x -> s:log('InsertLeave') }),
        \ )
    call callbag#pipe(
        \ callbag#empty(),
        \ callbag#subscribe({
        \   'next': {x->s:log('next will never be called')},
        \   'error': {e->s:log('error will never be called')},
        \   'complete': {->s:log('complete will be called')},
        \ }),
        \ )
    call callbag#pipe(
        \ callbag#never(),
        \ callbag#subscribe(
        \   {x->s:log('next will not be called')},
        \   {e->s:log('error will not be called')},
        \   {->s:log('complete will not be called')},
        \ ),
        \ )
     call callbag#pipe(
        \ callbag#create({next,error,done->next('next')}),
        \ callbag#subscribe({
        \   'next': {x->s:log('next')},
        \   'error': {e->s:log('error')},
        \   'complete': {->s:log('complete')},
        \ }),
        \ )
    call callbag#pipe(
        \ callbag#create({next,error,done->next('val')}),
        \ callbag#forEach({x->s:log('next value is ' . x)}),
        \ )
    call callbag#pipe(
        \ callbag#lazy({->2*10}),
        \ callbag#subscribe({
        \   'next':{x->s:log('next ' . x)},
        \   'complete': {->s:log('complete')},
        \ }),
        \ )
    call callbag#pipe(
        \ callbag#throwError('my dummy error'),
        \ callbag#subscribe({
        \   'next': {x->s:log('next will never be called')},
        \   'error': {e->s:log('error called with ' . e)},
        \   'complete': {->s:log('complete will never be called')},
        \ }),
        \ )
    call callbag#pipe(
        \ callbag#of(1, 2, 3, 4),
        \ callbag#subscribe({
        \   'next': {x->s:log('next value is ' . x)},
        \   'complete': {->s:log('completed of')},
        \ }),
        \ )
     call callbag#pipe(
        \ callbag#merge(
        \   callbag#fromEvent('InsertEnter'),
        \   callbag#fromEvent('InsertLeave'),
        \ ),
        \ callbag#forEach({x->s:log('InsertEnter or InsertLeave')}),
        \ )
     call callbag#pipe(
        \ callbag#fromEvent('TextChangedI', 'text_change_autocmd_group_name'),
        \ callbag#takeUntil(
        \   callbag#fromEvent('InsertLeave', 'insert_leave_autocmd_group_name'),
        \ ),
        \ callbag#debounceTime(250),
        \ callbag#subscribe({
        \   'next': {x->s:log('next')},
        \   'error': {x->s:log('error')},
        \   'complete': {->s:log('complete')},
        \ }),
        \ )
     call callbag#pipe(
        \ callbag#fromEvent('InsertEnter'),
        \ callbag#delay(2000),
        \ callbag#forEach({x->s:log('next')}),
        \ )
     call callbag#pipe(
        \ callbag#of(1,2,3,4,5,6,7,8,9),
        \ callbag#group(3),
        \ callbag#forEach({x->s:log(x)}),
        \ )
     call callbag#pipe(
        \ callbag#fromEvent('InsertEnter'),
        \ callbag#map({x->callbag#interval(1000)}),
        \ callbag#flatten(),
        \ callbag#forEach({x->s:log(x)}),
        \ )
     call callbag#pipe(
        \ callbag#of(1,2,3,4,5),
        \ callbag#scan({prev, x-> prev + x}, 0),
        \ callbag#forEach({x->s:log(x)}),
        \ )
    call callbag#pipe(
        \ callbag#concat(
        \  callbag#of(1,2,3),
        \  callbag#of(4,5,6),
        \ ),
        \ callbag#subscribe({
        \   'next':{x->s:log('next ' . x)},
        \   'complete': {->s:log('complete')},
        \ }),
        \ )
    call callbag#pipe(
        \ callbag#combine(
        \  callbag#interval(100),
        \  callbag#interval(350),
        \ ),
		\ callbag#take(10),
        \ callbag#subscribe({
        \   'next':{x->s:log('next '. x[0] . ' ' . x[1])},
        \   'complete': {->s:log('complete')},
        \ }),
        \ )
    call callbag#pipe(
		\ callbag#of(1, 2, 3, 4, 5),
		\ callbag#takeWhile({x -> x != 4}),
        \ callbag#subscribe({
        \   'next':{x->s:log('next ' . x)},
        \   'complete': {->s:log('complete')},
        \ }),
        \ )

    let l:MapAndTake3 = callbag#operate(
        \ callbag#map({x->x*10}),
        \ callbag#take(3),
        \ )
    call callbag#pipe(
        \ callbag#of(1,2,3,4,5,6,7,8,9),
        \ l:MapAndTake3,
        \ callbag#subscribe({
        \   'next': {x->s:log(x)},
        \   'complete': {->s:log('complete')},
        \ })
        \ )
    call callbag#pipe(
        \ callbag#fromArray([1,2,3,4]),
        \ callbag#subscribe({
        \   'next': {x->s:log(x)},
        \   'complete': {->s:log('complete')},
        \ }),
        \ )
endfunction
```

## Embedding

Please do not take direct dependency on this plugin and instead embed it using the following command.

```vim
:CallbagEmbed path=./autoload/myplugin/callbag.vim namespace=myplugin#callbag
```

This can then be referenced using `myplugin#callbag#pipe()`

## License

MIT

## Author

Prabir Shrestha
