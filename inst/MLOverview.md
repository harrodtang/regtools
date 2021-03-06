
#  The 5-Page Machine Learning Book

(The title here alludes to Andriy Burkov's excellent work,
*The Hundred-Page Machine Learning Book*.  Note too my own 
forthcoming book, *The Art of Machine Learning: Algorithms+Data+R*.)

Here we give an overview of the most widely used predictive methods in
statistical/machine learning.  For each one, we present

* background

* overview of how it works

* **qe***-series function

We'll also briefly discuss the overfitting issue.

## Notation

For convenience, we'll let Y denote the variable to be predicted, i.e.
the response variable, and let X denote the set of predictor
variables/features.

Our training data are (X<sub>1</sub>, Y<sub>1</sub>),.., (X<sub>n</sub>,
Y<sub>n</sub>).  We wish to predict new cases in the future, in which X
is known but Y needs to be predicted.

## Running example

The package's built-in dataset **mlb** consists of data on major league
baseball players.  

``` r
> data(mlb)
> head(mlb)
             Name Team       Position Height Weight   Age PosCategory
1   Adam_Donachie  BAL        Catcher     74    180 22.99     Catcher
2       Paul_Bako  BAL        Catcher     74    215 34.69     Catcher
3 Ramon_Hernandez  BAL        Catcher     72    210 30.78     Catcher
4    Kevin_Millar  BAL  First_Baseman     72    210 35.43   Infielder
5     Chris_Gomez  BAL  First_Baseman     73    188 35.71   Infielder
6   Brian_Roberts  BAL Second_Baseman     69    176 29.39   Infielder
```

## The  R package's **qe***-series functions

Here "qe" stands for **"quick and easy."**  The functions have a simple,
uniform interface, and most importantly, **require no setup.**  To fit
an SVM model, say, one simply calls **qeSVM()**, no preparation calls to
define the model etc.

The call
form is

``` r
model fit <- qe<model name>(<data name>,<Y name>)
```

As noted, no prior calls are needed to define the model, etc.

## Example

Let's predict weight from height and age, using two methods, k-Nearest
Neighbor and random forests.

``` r
mlb <- mlb[,4:6]  # columns for height, weight and age
knnout <- qeKNN(mlb,'Weight')  # fit k-Nearest Neighbor model
rfout <- qeRF(mlb,'Weight')  # fit random forests model
```

Default values of hyperparameters are used but can be overridden.

Prediction of new cases is equally easy, in the form

``` r
predict(<model fit>, <new X value>)
```

E.g. to predict the weight of a new player of height 70 and age 28, run

``` r
> predict(rfout,c(70,28))
       2 
184.1626 
```

The model is automatically assessed on a holdout set (the concept is
discussed later in this document):

``` r
> rfout$testAcc  # mean absolute prediction error
[1] 15.16911
```

(The formation of a holdout set can be suppressed.)

Each **qe** function sets default values for the *tuning parameters*
or *hyperparameters* for the given ML method.  Nondefault values,
specific to the given ML, can optionally be specified.

## Regression and classification problems, regression functions

Prediction applications in which Y is a continuous variable, say weight,
or at least ordinal, are called *regression settings*.  Applications in
which Y is categorical, i.e. Y is a factor variable in R, say predicting
the player's position (e.g. pitcher) are *classification settings*.

Somewhat confusingly, both settings make use of the *regression function*,
m(t) = E(Y | X = t), the mean value of Y in the subpopulation defined by
X = t.  If say we are predicting weight in the **mlb** data, then for instance
m(71,28) would be the mean weight among all players of height 71 inches
and 28 years old.  To predict the weight of a new player, say height 77
and age 19, we use m(77,19).

In classification problems, Y is converted to a set of indicator
variables.  For the position 'pitcher' in the **mlb** data, we would have
Y = 1 or 0, depending on whether the player is a pitcher or not.
(Position is in column 3 of the original dataset.)
Then  E(Y | X = t) reduces to P(Y = 1 | X = t), the probability that the
player is a pitcher given the player's height, weight and age, say.  We
would typically find probabilities for each position, then guess the one
with the highest probability.

In other words, the regression function m(t) is central to both regression
and classification settings.  The statistical/machine learning methods
presented here amount to ways to estimate m(t).  The methods are
presented below in an order that shows connection between them.

Even though each ML method has its own special *tuning parameters* or
*hyperparameters*, used to fine-tune performance, they all center around
the regression function m(t).

The **qe** series function sense whether the user is specifying a
regression setting or a classification setting, by noting whether the Y
variable (second argument) is numeric or an R factor.

## ML predictive methods

We now present the "30,000 foot" view of the major statistical/machine
learning methods.

### k-Nearest Neighbors

This method was originally developed by statisticians, starting in the 1950s
and 60s.

It's very intuitive.  To predict, say, the weight of a new
player of height 72 and age 25, we find the k closest players in our 
training data to (72,25), and average their weights.  This is our
estimate of m(72,25), and we use it as our prediction.

The **qeKNN()** function wraps **kNN()** in **regtools**.  The main
hyperparameter is the number of neighbors.

### Random forests

This method was developed mainly by statisticians, starting in the
1980s.

This is a natural extension of k-NN, in that it too creates a
neighborhood and averages Y values within the neighborhood.  However, it
does so in a different way, creating tree structures.

Say we are predicting player position in the **mlb** data, from height,
weight and age.  (E.g. catchers tend to be heavier and older.)  We first
ask whether the height is above or below a certain threshold.  After
that, we ask whether weight is above or below a certain (different)
threshold.  This creates a "box" in height-weight space.  We then might
subdivide the box according to whether age is above or below a
threshold.

The word *might* in the last sentence alludes to the fact that the
process may stop early, if the current subdivision is judged fine enough
to produce good accuracy.  And one generally wants to avoid having
neighborhoods (*nodes* in the tree) that don't have many data points;
this is controlled by a hyperparameter.

Clearly, the order in which the predictor variables are evaluated
(height, weight and age above) can matter a lot.  So, more than one tree
is constructed, with random orders.  The number of trees is another
hyperparameter.

The **qeRF()** function wraps the function of the same name in the
**randomForests** package.

### Boosting

This method has been developed both by CS and statistics people.  The
latter have been involved mainly in gradient boosting, the technique
used here.

The basic idea is to iteratively build up a sequence of trees, each of which
is focused on the data points on which the last predicted poorly.  At
the end, all the trees are combined.

The **qeGBoost()** wraps **gbm()** in the package of the same name.  It
is tree-based, with hyperparameters similar to the random forests case,
plus a *learning rate*.  The latter controls the size of iteration
steps.

### Linear model

This of course is the classical linear regression model, invented
200 years ago (!) and developed by statisticians.

For example, a model for mean weight, given height and age, would be

m(height,age) = 
&beta;<sub>0</sub> +
&beta;<sub>1</sub> height + 
&beta;<sub>2</sub> age

for unknown population constants &beta;<sub>i</sub>, which are estimated
from our training data.

This model is mainly for regression settings, though some analysts use
it in classification.  If used in conjunction with polynomials (see
below), this may work as well or better than the logistic model (see
below).

The function **qeLin()** wraps the ordinary **lm()**.  It mainly
just calls the latter, but does some little fixess.

### Logistic model

This is a generalization of the linear model, developed by
statisticians.

This model is only for classification settings.  Since m(t) is now a
probability, we need it to have values in the interval [0,1].  This is
achieved by feeding a linear model into the *logistic function*,
l(u) = (1 + exp(-u))<sup>-1</sup>.  So for instance, to predict whether a
player is a catcher (Y = 1 if yes, Y = 0 if no),

P(catcher | height, weight, age) = m(height,weight,age) = 
1 / [1 + exp{-(&beta;<sub>0</sub> +
&beta;<sub>1</sub> height + 
&beta;<sub>2</sub> weight +
&beta;<sub>3</sub> age)}]

The function **qeLogit()** wraps the ordinary R function **glm()**, but
adds an important feature:  **glm()** only handles the 2-class setting,
e.g.  catcher vs. non-catcher.  The **qeLlogit()** handles the c-class
situation by calling **glm()** one class at a time, generating c
**glm()** outputs.  When a new case is to be predicted, it is fed into
each of the c **glm()** outputs, yielding c probabilities.  It then
predicts the new case as whichever class has the highest probability.

### Polynomial-linear models

Some people tend to shrink when they become older.  Thus we may wish to
model a tendency for people to gain weight in middle age but then lose
weight as seniors, say 

m(height,age) = 
&beta;<sub>0</sub> +
&beta;<sub>1</sub> height + 
&beta;<sub>2</sub> age +
&beta;<sub>3</sub> age<sup>2</sup>

where presumably &beta;<sub>3</sub> < 0.

We may even include a height X age product term, allowing for interactions.
Polynomials of degree 3 and so on could also be considered.

This would seem nonlinear, but that would be true only in the sense of
being nonlinear in age.  But it is still linear in the
&beta;<sub>i</sub>, so **qeLin()** can be used, or **qeLogit()**
for classification settings.

Forming the polynomial terms by hand would be tedious, especially since
we would also have to do this for predicting new cases.  Instead, we use
**qePolyLin()** (regression setting) and **qePolyLog()**
(classification).  They make use of the package **polyreg**.

Polynomial models can in many applications hold their own with the fancy
ML methods.  One must be careful, though, about overfitting.

### The LASSO

Some deep mathematical theory implies that in linear models it may be
advantageous to shrink the &beta;<sub>i</sub>.  The LASSO method does
this in a mathematically rigorous manner.  The LASSO is especially
popular as a tool for predictor variable selection (see below).

The function **qeLASSO()** wraps **cvglmnet()** in the **glmnet**
package.  The main hyperparameter controls the amount of shrinkage.

### Support Vector Machines

These were developed originally in the AI community, and later attracted
interest among statisticians.  They are used mainly in classification
settings.

Say we are predicting catcher vs. non-catcher, based on height and
weight.  We might plot a scatter diagram, with height on the horizontal
axis and weight on the vertical, using red dots for the catchers and
blue dots for the non-catchers.  We might draw a line that best separates
the red and blue dots, then predict new cases by observing which side of
the line they fall on.  This is what SVM does (with more
predictors the line become a plane or hyperplane).

The **qeSVM()** function wraps **svm()** in the **e1071** package.  What
about hyperparameters?  Here are the two main ones:

* **gamma:** This is polynomial degree or another similar criterion.

* **cost:** Ideally, SVM will cleanly separate the data points in the
  training set by class, e.g. all red dots on one side of the line and
all blue on the other.  But for most datasets, this will not occur.  The
cost variable is roughly saying how many exceptions we are willing to
accept.

### Neural networks

These were developed almost exclusively in the AI community.

An NN consists of one or more *layers*, each of which consists of a
number of *neurons*.  Say for concreteness we have 10 neurons per layer.
The output of the first layer will be 10 linear combinations of the
predictor variables/features, essentially 10 linear regression models.
Those will be fed into the second layer, yielding 10 "linear
combinations of linear combinations," and so on.

In regression settings, the outputs of the last layer will be averaged
together to produce our estimated m(t).  In the classification case with
c classes, our final layer will have c outputs; whichever is largest
will be our predicted class.

NNs are arguably the most complex of all the methods described here, and
tend to have many -- even millions -- of hyperparameters, stemming in
large part because the numbers of layers and neurons per layer may be
quite large.  The coefficients in all those linear combinations
(*weights*) are computed iteratively.

The **qeNeural()** function allows specifying the numbers of layers and
neurons per layer, and the number of iterations.  It wraps **krsFit()**
from the **regtools** package, which in turn wraps the R **keras**
package (and there are further wraps involved after that).

## Overfitting

Up to a point, the more complex a model is, the greater its predictive
power.  "More complex" can mean adding more predictor variables, using a
higher-degree polynomial, adding more layers etc.

But "the Law of Diminishing Returns" takes hold eventually, and
accuracy will decline.  At the latter point we say we've overfit.
Though there is currently some controversy on this, it is a key point to
consider.

All the **qe** functions allow one to randomly partition the data,
treating one part as the training set, and using the rest, the *holdout*
set, to treat as new data to get a reliable estimate of the performance
of the model; the function fits on the training set, then uses the
result to predict the holdout set.  Though a model may predict well on
the training set, it may do less well on the holdout data, indicating
overfitting.

Among other things, this means that if our training data have a very
large number of predictor variables, we may wish to delete some, or
possibly combine them, in order to reduce complexity.  

One way to combine them is *Principal Components Analysis* (PCA).  One
creates new predictor variables as linear combinations of the original
ones, then retains only the combinations that have a large variance.  As
you may have guessed by now, "There's an app for that" -- **qePCA()**.

Many people use the LASSO for predictor variable selection.  Since
typically most of the LASSO coefficients are 0s, we use only the
variables with nonzero coefficients.  One would then predict with that
simplified LASSO model.

## Which one to use?

With many datasets, all the above ML methods will give similar results.
But for some other datasets, it really makes a difference which one we
use.

So, we should try all of methods of interest with holdout data, then
compare.  Well, "there's an app" for this too, **qeCompare()**.


