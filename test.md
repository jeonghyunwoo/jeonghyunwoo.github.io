## How to use `python::scikit-learn` as `caret package` of R
### pre-requisites
 - python
 - scikit-learn package
 * anaconda make to install python and its friends easy
I'll make examples using `recipes::credit_data`.
`recipes` are so very cool package for preprocessing.
### packages loading
I use `pacman` for easy loading of required packages.
```
library(pacman) # needed for 'p_load'
p_load(plyr,tidyverse,recipes,reticulate,resample)
```
### data loading and preprocess
```
data(credit_data)
df <- credit_data %>% rename_all(tolower)
# missing values check
sapply(df,function(x) sum(is.na(x)))

# preprocessing
df <- recipe(status~., data=df) %>% 
  step_meanimpute(all_numeric()) %>% 
  step_modeimpute(all_nominal()) %>% 
  step_center(all_numeric()) %>% 
  step_scale(all_numeric()) %>% 
  step_dummy(all_nominal(),-status) %>% 
  step_zv(all_predictors()) %>% 
  prep(training=df,retain=TRUE) %>% 
  juice()
 
 # missing values check
 sapply(df,function(x) sum(is.na(x)))
 ```
 ### data spliting
 ```
set.seed(2474) # for repex
splt <- initial_split(df,prop=0.7) # rsample
tr <- training(splt)
te <- testing(splt)

# for sklearn fitting
pd <-import('pandas')
x_train <-pd$DataFrame(dict(select(tr, -status)))
y_train <-ifelse(tr$status=='bad',1,0) %>% as.array
x_test <-pd$DataFrame(dict(select(te, -status)))
y_test <-ifelse(te$status=='bad',1,0) %>% as.array
```
### warm up: One model fitting (using scikit-learn)
```
ens <- import('sklearn.ensemble')
rf <- ens$RandomForestClassifier(n_estimators=100L, n_jobs=-1L)
rf$fit(x_train, y_train)
# test score
rf$score(x_test, y_test)
# predict
pred <-rf$predict(x_test)
prob <-rf$predict_proba(x_test)
```
### Two model fitting at once
```
sk <- import('sklearn')
two_model <-list(rf  = sk$ensemble$RandomForestClassifier,
                 svm = sk$svm$SVC) %>%
            map(~.$fit(x_train,y_train))
# predict
fit_df <- tibble(algo  = names(two_model), 
                 model = two_model) %>%
          mutate(pred = map(model, ~.$predict(x_test) %>% as.vector) )
```
------------------------------

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```
