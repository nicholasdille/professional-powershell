<!-- .slide: id="scope" -->

## Scope

The guidelines apply to scripts as well as modules

They do not apply to commands written on the console

--

<!-- .slide: id="code_reuse" -->

## Code reuse

You should always develop for code reuse

Bad style: Monolithic script which runs all or nothing

Good style: Separate code into functions

Also increases testability

--

<!-- .slide: id="console_use" -->

## Develop for the Console

Look at the `*-Item` comdlets as a design example

Use [advanced functions](#/advanced_functions)

Add [comment based help](#/get_help)

Use the [pipeline](#/pipelines)