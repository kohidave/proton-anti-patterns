# Non Parameterized Templates

I sometimes see folks who aren't aware that in Proton, you can parameterize templates so that the same template can be used several times. In Proton-land, we think of our templates almost like Classes in object-oriented-programing. Proton templates define a _schema_ which defines the inputs to collect and use when rendering that template. Template authors can then use those inputs to parameterize their templates. 

In this repo, you'll see two examples of templates. The `anti-pattern` template, which uses one template per service instance that we're deploying to - and the simplified, parameterized version, which uses just one template and collects inputs from template users to dynamically render. 

I've also included some sample specs - specs are the place where developers provide the inputs to a template. 