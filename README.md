Ruby encoding error triggered by Parade
=======================================

This is a [Parade](http://rubygems.org/gems/parade) slide deck that triggers an
encoding error in Ruby. To reproduce the error, clone this repo and run:

```
$ bundle install
$ LANG= LC_ALL= bundle exec parade
```

Then visit `http://localhost:9090/` and you should see an error page saying
"ArgumentError at /: invalid byte sequence in US-ASCII". The error is reported
as coming from `parade-0.10.2/lib/parade/parsers/dsl.rb:36`, where
`eval(contents)` is called. If you log what `contents` is, you'll see it's the
content of the `parade` file with encoding `US-ASCII`, which is the encoded
assumed if `LANG` and `LC_ALL` are unset.

This file `parade` only contains bytes less than `0x80` and is thus a valid
`US-ASCII` string. The invalid file is actually `hello/hello.md`, which contains
the byte sequence `0xC2 0xA0`, which is a UTF-8 encoded U+00A0, a non-breaking
space and not a valid US-ASCII sequence.

So, Ruby is blowing up when trying to `eval` a valid string of Ruby code,
because an invalidly encoded string of Markdown code also exists. Presumably the
Markdown code has not gone through `eval`, it goes through the slide parser. I'm
not sure why its presence makes `eval` fall over, all I know is Parade triggers
this in some way.
