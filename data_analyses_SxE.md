data\_analyses\_SxE
================
Rebecca Batstone
2019-09-03

Load packages
-------------

``` r
# packages
library("tidyverse") ## includes ggplot2, dplyr, readr, stringr
library("cowplot") ## paneled graphs
library("car") ## Anova function
```

Load raw means
--------------

``` r
# created using "data_analyses_raw_means.Rmd"

# mean across environments
load("./raw_means/G_combined_raw.Rdata")
## mean within each environment:
load("./raw_means/GxE_combined_raw.Rdata")
```

Set to effects contrasts
------------------------

``` r
options(contrasts = rep ("contr.sum", 2)) 
```

Option 1: global means (as used in first version of the manuscript)
-------------------------------------------------------------------

### Calculate rel/stand means

``` r
# calculate relativized and standardized traits
GE_comb_raw_sel <- GE_comb_raw %>%
  mutate(rel_surv = surv/mean(surv, na.rm=TRUE),
         rel_shoot = shoot/mean(shoot, na.rm=TRUE),
         rel_leaf = leaf/mean(leaf, na.rm=TRUE),
         rel_flower = flower_succ/mean(flower_succ, na.rm=TRUE),
         rel_fruit = fruits/mean(fruits, na.rm=TRUE),
         rel_fruit_succ = fruit_succ/mean(fruit_succ, na.rm=TRUE),
         stand_nod = (nod - mean(nod, na.rm=TRUE))/sd(nod, na.rm=TRUE)) %>%
  as.data.frame(.)
```

### Option 1: S x E models

``` r
SxE_function <- function(df, trait){
   print(paste(trait))
   lin_mod <- lm(get(trait) ~ stand_nod*env, data = df)
   lin_mod_sum <- summary(lin_mod)
   lin_mod_aov <- Anova(lin_mod, type = 3)
   quad_mod <- lm(get(trait) ~ (stand_nod + I(stand_nod^2))*env, data = df)
   quad_mod_sum <- summary(quad_mod)
   quad_mod_aov <- Anova(quad_mod, type = 3)
   mod_comp <- anova(lin_mod, quad_mod)
   
   return(list(lin_mod_sum, quad_mod_sum, lin_mod_aov, quad_mod_aov, mod_comp))
 }
 
# specify vars to loop over
var.list <- c("rel_surv","rel_shoot","rel_leaf","rel_flower","rel_fruit","rel_fruit_succ")
 
SxE_out <- lapply(var.list, SxE_function, df = GE_comb_raw_sel)
```

    ## [1] "rel_surv"
    ## [1] "rel_shoot"
    ## [1] "rel_leaf"
    ## [1] "rel_flower"
    ## [1] "rel_fruit"
    ## [1] "rel_fruit_succ"

Option 2: means within envs (as suggested by R1)
------------------------------------------------

### Calculate rel/stand means

``` r
# calculate relativized and standardized traits
GE_comb_raw_sel2 <- GE_comb_raw %>%
  group_by(env) %>%
  mutate(rel_surv = surv/mean(surv, na.rm=TRUE),
         rel_shoot = shoot/mean(shoot, na.rm=TRUE),
         rel_leaf = leaf/mean(leaf, na.rm=TRUE),
         rel_flower = flower_succ/mean(flower_succ, na.rm=TRUE),
         rel_fruit = fruits/mean(fruits, na.rm=TRUE),
         rel_fruit_succ = fruit_succ/mean(fruit_succ, na.rm=TRUE),
         stand_nod = (nod - mean(nod, na.rm=TRUE))/sd(nod, na.rm=TRUE)) %>%
  as.data.frame(.)
```

### Option 2: S x E models

``` r
SxE_out2 <- lapply(var.list, SxE_function, df = GE_comb_raw_sel2)
```

    ## [1] "rel_surv"
    ## [1] "rel_shoot"
    ## [1] "rel_leaf"
    ## [1] "rel_flower"
    ## [1] "rel_fruit"
    ## [1] "rel_fruit_succ"

### Option 3: standardize traits across environments, absolute fitness

``` r
# new var list (absolute fitness values
var.list2 <- c("surv","shoot","leaf","flower_succ","fruits","fruit_succ")
SxE_out3 <- lapply(var.list2, SxE_function, df = GE_comb_raw_sel)
```

    ## [1] "surv"
    ## [1] "shoot"
    ## [1] "leaf"
    ## [1] "flower_succ"
    ## [1] "fruits"
    ## [1] "fruit_succ"

### Option 4: standardize nodules globally, relativize fitness within environment

``` r
# calculate relativized and standardized traits
GE_comb_raw_sel4 <- GE_comb_raw %>%
  mutate(stand_nod = (nod - mean(nod, na.rm=TRUE))/sd(nod, na.rm=TRUE)) %>%
  group_by(env) %>%
  mutate(rel_surv = surv/mean(surv, na.rm=TRUE),
         rel_shoot = shoot/mean(shoot, na.rm=TRUE),
         rel_leaf = leaf/mean(leaf, na.rm=TRUE),
         rel_flower = flower_succ/mean(flower_succ, na.rm=TRUE),
         rel_fruit = fruits/mean(fruits, na.rm=TRUE),
         rel_fruit_succ = fruit_succ/mean(fruit_succ, na.rm=TRUE)) %>%
  as.data.frame(.)
```

### Option 4: S x E models

``` r
SxE_out4 <- lapply(var.list, SxE_function, df = GE_comb_raw_sel4)
```

    ## [1] "rel_surv"
    ## [1] "rel_shoot"
    ## [1] "rel_leaf"
    ## [1] "rel_flower"
    ## [1] "rel_fruit"
    ## [1] "rel_fruit_succ"

### Combine results

``` r
SxE_lin_aov1 <- rbind(SxE_out[[1]][[3]], SxE_out[[2]][[3]], SxE_out[[3]][[3]], ## global all
                     SxE_out[[4]][[3]],SxE_out[[5]][[3]],SxE_out[[6]][[3]])

SxE_quad_aov1 <- rbind(SxE_out[[1]][[4]], SxE_out[[2]][[4]], SxE_out[[3]][[4]], ## global all
                     SxE_out[[4]][[4]],SxE_out[[5]][[4]], SxE_out[[6]][[4]])

SxE_lin_aov1a <- rbind(SxE_out3[[1]][[3]], SxE_out3[[2]][[3]], SxE_out3[[3]][[3]], ## abs fit, global nod
                     SxE_out3[[4]][[3]],SxE_out3[[5]][[3]], SxE_out3[[6]][[3]])

SxE_quad_aov1a <- rbind(SxE_out3[[1]][[4]], SxE_out3[[2]][[4]], SxE_out3[[3]][[4]], ## abs fit, global nod
                     SxE_out3[[4]][[4]],SxE_out3[[5]][[4]], SxE_out3[[6]][[4]])

SxE_lin_aov2 <- rbind(SxE_out2[[1]][[3]], SxE_out2[[2]][[3]], SxE_out2[[3]][[3]], ## within all
                     SxE_out2[[4]][[3]],SxE_out2[[5]][[3]], SxE_out2[[6]][[3]])

SxE_quad_aov2 <- rbind(SxE_out2[[1]][[4]], SxE_out2[[2]][[4]], SxE_out2[[3]][[4]], ## within all
                     SxE_out2[[4]][[4]],SxE_out2[[5]][[4]], SxE_out2[[6]][[4]])

SxE_lin_aov3 <- rbind(SxE_out4[[1]][[3]], SxE_out4[[2]][[3]], SxE_out4[[3]][[3]], ## within fit, global nod
                     SxE_out4[[4]][[3]],SxE_out4[[5]][[3]], SxE_out4[[6]][[3]])

SxE_quad_aov3 <- rbind(SxE_out4[[1]][[4]], SxE_out4[[2]][[4]], SxE_out4[[3]][[4]], ## within fit, global nod
                     SxE_out4[[4]][[4]],SxE_out4[[5]][[4]], SxE_out4[[6]][[4]])

SxE_lin_comp <- cbind(SxE_lin_aov1, SxE_lin_aov1a,
                      SxE_lin_aov2, SxE_lin_aov3) ## compare global models to within-environment models
SxE_quad_comp <- cbind(SxE_quad_aov1, SxE_quad_aov1a,
                       SxE_quad_aov2, SxE_quad_aov3) ## compare global models to within-environment models

write.csv(SxE_lin_comp, "./SxE_analyses_outputs/SxE_res_lin_comp.csv")
write.csv(SxE_quad_comp, "./SxE_analyses_outputs/SxE_res_quad_comp.csv")
```

Selection coefficients/gradients within each environment
--------------------------------------------------------

``` r
# function to iterate over environments, and traits

sel_win_env_function <- function(df, trait){
   trait <- print(paste(trait))  
   lin_mod <- lm(get(trait) ~ stand_nod, data = df)
   lin_mod_sum <- summary(lin_mod)
   lin_mod_aov <- Anova(lin_mod, type = 2)
   quad_mod <- lm(get(trait) ~ stand_nod + I(stand_nod^2), data = df)
   quad_mod_sum <- summary(quad_mod)
   quad_mod_aov <- Anova(quad_mod, type = 2)
   mod_comp <- anova(lin_mod, quad_mod)
   
   return(list(trait, lin_mod_sum, quad_mod_sum, lin_mod_aov, quad_mod_aov, mod_comp))
}

# apply to traits with data in all envs

# assign vars to loop over
env.list1 <- unique(GE_comb_raw_sel$env)

sel_out1 <- lapply(env.list1, FUN = function(e){
  df.use <- filter(GE_comb_raw_sel, env==e)
  surv.out <- sel_win_env_function(df.use, var.list[1])
  shoot.out <- sel_win_env_function(df.use, var.list[2])
  leaf.out <- sel_win_env_function(df.use, var.list[3])
  env <- print(paste(e))  
  
  return(list(env, surv.out, shoot.out, leaf.out))
})
```

    ## [1] "rel_surv"
    ## [1] "rel_shoot"
    ## [1] "rel_leaf"
    ## [1] "GH"
    ## [1] "rel_surv"
    ## [1] "rel_shoot"
    ## [1] "rel_leaf"
    ## [1] "plot_1"
    ## [1] "rel_surv"
    ## [1] "rel_shoot"
    ## [1] "rel_leaf"
    ## [1] "plot_2"
    ## [1] "rel_surv"
    ## [1] "rel_shoot"
    ## [1] "rel_leaf"
    ## [1] "plot_3"
    ## [1] "rel_surv"
    ## [1] "rel_shoot"
    ## [1] "rel_leaf"
    ## [1] "plot_4"

``` r
# apply to traits with data in only some envs

# assign vars to loop over
env.list2 <- c("plot_1","plot_2","plot_3")

sel_out2 <- lapply(env.list2, FUN = function(e){
  df.use <- filter(GE_comb_raw_sel, env==e)
  flower.out <- sel_win_env_function(df.use, var.list[4])
  fruit.out <- sel_win_env_function(df.use, var.list[5])
  fruit_succ.out <- sel_win_env_function(df.use, var.list[6])
  env <- print(paste(e))  
  
  return(list(env, flower.out, fruit.out, fruit_succ.out))
})
```

    ## [1] "rel_flower"
    ## [1] "rel_fruit"
    ## [1] "rel_fruit_succ"
    ## [1] "plot_1"
    ## [1] "rel_flower"
    ## [1] "rel_fruit"
    ## [1] "rel_fruit_succ"
    ## [1] "plot_2"
    ## [1] "rel_flower"
    ## [1] "rel_fruit"
    ## [1] "rel_fruit_succ"
    ## [1] "plot_3"

``` r
# Combine dfs (ANOVA res)
ANOVA_sel_survival <- rbind(sel_out1[[1]][[2]][[4]],sel_out1[[1]][[2]][[5]], ## GH
                            sel_out1[[2]][[2]][[4]],sel_out1[[2]][[2]][[5]], ## plot_1
                            sel_out1[[3]][[2]][[4]],sel_out1[[3]][[2]][[5]], ## plot_2
                            sel_out1[[4]][[2]][[4]],sel_out1[[4]][[2]][[5]], ## plot_3
                            sel_out1[[5]][[2]][[4]],sel_out1[[5]][[2]][[5]]) ## plot_4

ANOVA_sel_shoot <- rbind(sel_out1[[1]][[3]][[4]],sel_out1[[1]][[3]][[5]], 
                         sel_out1[[2]][[3]][[4]],sel_out1[[2]][[3]][[5]],
                         sel_out1[[3]][[3]][[4]],sel_out1[[3]][[3]][[5]], 
                         sel_out1[[4]][[3]][[4]],sel_out1[[4]][[3]][[5]],
                         sel_out1[[5]][[3]][[4]],sel_out1[[5]][[3]][[5]])

ANOVA_sel_leaf <- rbind(sel_out1[[1]][[4]][[4]],sel_out1[[1]][[4]][[5]], 
                        sel_out1[[2]][[4]][[4]],sel_out1[[2]][[4]][[5]],
                        sel_out1[[3]][[4]][[4]],sel_out1[[3]][[4]][[5]], 
                        sel_out1[[4]][[4]][[4]],sel_out1[[4]][[4]][[5]],
                        sel_out1[[5]][[4]][[4]],sel_out1[[5]][[4]][[5]])

ANOVA_sel_flower <- rbind(sel_out2[[1]][[2]][[4]],sel_out2[[1]][[2]][[5]], 
                          sel_out2[[2]][[2]][[4]],sel_out2[[2]][[2]][[5]],
                          sel_out2[[3]][[2]][[4]],sel_out2[[3]][[2]][[5]])

ANOVA_sel_fruit <- rbind(sel_out2[[1]][[3]][[4]],sel_out2[[1]][[3]][[5]], 
                         sel_out2[[2]][[3]][[4]],sel_out2[[2]][[3]][[5]],
                         sel_out2[[3]][[3]][[4]],sel_out2[[3]][[3]][[5]])

ANOVA_sel_fruit_succ <- rbind(sel_out2[[1]][[4]][[4]],sel_out2[[1]][[4]][[5]], 
                         sel_out2[[2]][[4]][[4]],sel_out2[[2]][[4]][[5]],
                         sel_out2[[3]][[4]][[4]],sel_out2[[3]][[4]][[5]])

ANOVA_sel_comb1 <- cbind(ANOVA_sel_survival, ANOVA_sel_shoot, ANOVA_sel_leaf)
ANOVA_sel_comb2 <- cbind(ANOVA_sel_flower, ANOVA_sel_fruit, ANOVA_sel_fruit_succ)
ANOVA_sel_all <- rbind(ANOVA_sel_comb1, ANOVA_sel_comb2)
write.csv(ANOVA_sel_all, "./SxE_analyses_outputs/ANOVA_sel_win_env.csv")

# combine dfs (regression res)
reg_sel_survival <- rbind(sel_out1[[1]][[2]][[2]]$coefficients, ## GH
                            sel_out1[[2]][[2]][[2]]$coefficients, ## plot_1
                            sel_out1[[3]][[2]][[2]]$coefficients, ## plot_2
                            sel_out1[[4]][[2]][[2]]$coefficients, ## plot_3
                            sel_out1[[5]][[2]][[2]]$coefficients) ## plot_4

reg_sel_shoot <- rbind(sel_out1[[1]][[3]][[2]]$coefficients, 
                         sel_out1[[2]][[3]][[2]]$coefficients,
                         sel_out1[[3]][[3]][[2]]$coefficients, 
                         sel_out1[[4]][[3]][[2]]$coefficients,
                         sel_out1[[5]][[3]][[2]]$coefficients)

reg_sel_leaf <- rbind(sel_out1[[1]][[4]][[2]]$coefficients, 
                        sel_out1[[2]][[4]][[2]]$coefficients,
                        sel_out1[[3]][[4]][[2]]$coefficients, 
                        sel_out1[[4]][[4]][[2]]$coefficients,
                        sel_out1[[5]][[4]][[2]]$coefficients)

reg_sel_flower <- rbind(sel_out2[[1]][[2]][[2]]$coefficients, 
                          sel_out2[[2]][[2]][[2]]$coefficients,
                          sel_out2[[3]][[2]][[2]]$coefficients)

reg_sel_fruit <- rbind(sel_out2[[1]][[3]][[2]]$coefficients, 
                         sel_out2[[2]][[3]][[2]]$coefficients,
                         sel_out2[[3]][[3]][[2]]$coefficients)

reg_sel_fruit_succ <- rbind(sel_out2[[1]][[4]][[2]]$coefficients, 
                         sel_out2[[2]][[4]][[2]]$coefficients,
                         sel_out2[[3]][[4]][[2]]$coefficients)

reg_sel_comb1 <- cbind(reg_sel_survival, reg_sel_shoot, reg_sel_leaf)
reg_sel_comb2 <- cbind(reg_sel_flower, reg_sel_fruit, reg_sel_fruit_succ)
reg_sel_all <- rbind(reg_sel_comb1, reg_sel_comb2)
write.csv(reg_sel_all, "./SxE_analyses_outputs/reg_sel_win_env.csv")
```

Plots for selection analyses
----------------------------

``` r
# plots

env_names <- list(
  'GH'="Greenhouse",
  'plot_1'="Plot 1",
  'plot_2'="Plot 2",
  'plot_3'="Plot 3",
  'plot_4'="Plot 4"
)

env_labeller <- function(variable,value){
  return(env_names[value])
}

# assign colours to plots
plot_colours <- c('green4','firebrick3', 'darkblue','darkorchid', 'darkgoldenrod')
names(plot_colours) <- levels(GE_comb_raw_sel$env)
colScale <- scale_colour_manual(name = "Environment", values = plot_colours, labels = env_labeller)

# define colours for plots
GH <- "green4"
plot_1 <- "firebrick3"
plot_2 <- "darkblue"
plot_3 <- "darkorchid"
plot_4 <- "darkgoldenrod"

# create plotting function for facetted plots across environments

# shoot

plot_sel_all_fun <- function(df, trait){

(plot <- ggplot(data=df, aes(x=stand_nod, y=get(trait), colour= env)) +
  geom_smooth(method=lm, se=FALSE, aes(colour = env), formula = y ~ x, linetype = 1) +
  geom_point(size=4) +
  theme_bw() + 
  colScale +  
  xlab(NULL) + 
  ylab(paste(trait)) +
  theme(axis.title.y = element_text(colour = "black", size = 24), 
        axis.text.y = element_text(size=20), 
        axis.title.x = element_text(colour = "black", size = 24), 
        axis.text.x = element_text(size=20), 
        strip.text = element_text(size=14, face="bold"),
        legend.position="none",
        legend.title = element_blank(),
        legend.text = element_text(colour="black", size=18),
        legend.background = element_blank(),
        plot.title = element_text(hjust=0.5, size=16, face = "bold"),
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank())) 
  
  return(plot)
  
}

rel_traits <- c("rel_surv","rel_shoot","rel_leaf","rel_flower","rel_fruit","rel_fruit_succ")
abs_traits <- c("surv","shoot","leaf","flower_succ","fruits","fruit_succ")

plot_sel_all_op1 <- lapply(abs_traits, plot_sel_all_fun, df = GE_comb_raw_sel) ## absolute fitness, global nods
plot_sel_all_op2 <- lapply(rel_traits, plot_sel_all_fun, df = GE_comb_raw_sel) ## global fitness, global nods
plot_sel_all_op3 <- lapply(rel_traits, plot_sel_all_fun, df = GE_comb_raw_sel2) ## within fitness, within nods
plot_sel_all_op4 <- lapply(rel_traits, plot_sel_all_fun, df = GE_comb_raw_sel4) ## within fitness, global nods

# plot specifications
abs_surv <- plot_sel_all_op1[[1]] + ylab("Absolute \n survival")
abs_shoot <- plot_sel_all_op1[[2]] + ylab("Absolute \n shoot biomass")
abs_leaf <- plot_sel_all_op1[[3]] + ylab("Absolute \n leaves") + xlab("Globally-standardized \n nodules")

glob_all_surv <- plot_sel_all_op2[[1]] + ylab("Globally-relativized \n survival")
glob_all_shoot <- plot_sel_all_op2[[2]] + ylab("Globally-relativized \n shoot biomass")
glob_all_leaf <- plot_sel_all_op2[[3]] + ylab("Globally-relativized \n leaves") + xlab("Globally-standardized \n nodules")

win_all_surv <- plot_sel_all_op3[[1]] + ylab("Locally-relativized \n survival")
win_all_shoot <- plot_sel_all_op3[[2]] + ylab("Locally-relativized \n shoot biomass")
win_all_leaf <- plot_sel_all_op3[[3]] + ylab("Locally-relativized \n leaves ") + xlab("Locally-standardized \n nodules")

win_fit_surv <- plot_sel_all_op4[[1]] + ylab("Locally-relativized \n survival")
win_fit_shoot <- plot_sel_all_op4[[2]] + ylab("Locally-relativized \n shoot biomass")
win_fit_leaf <- plot_sel_all_op4[[3]] + ylab("Locally-relativized \n leaves ") + xlab("Globally-standardized \n nodules")

# cowplots

# put all four plots into one
fig_base <- plot_grid(abs_surv, glob_all_surv, win_all_surv, win_fit_surv,
                      abs_shoot, glob_all_shoot, win_all_shoot, win_fit_shoot,
                      abs_leaf, glob_all_leaf, win_all_leaf, win_fit_leaf,
          ncol = 4,
          nrow = 3,
          align = "hv",
          labels = NULL)

save_plot("./figures/FigS1_SxE_all.pdf", fig_base,
          ncol = 4, # we're saving a grid plot of 2 columns
          nrow = 3, # and 3 rows
          # each individual subplot should have an aspect ratio of 1.3
          base_aspect_ratio = 1.3
          )

# Within-env selection
plot_sel_win_fun <- function(df, envir){
  
  env.out<-paste(envir)
  
   df.env <- df %>%
   filter(env == envir) %>%
   droplevels(.)
  
   df.lm <- df.env[ , c("rel_shoot", "stand_nod")]
   names(df.lm) <- c("y", "x")

(plot <- ggplot(data = df.env, aes(x=stand_nod, y=rel_shoot)) +
  geom_point(size=4, colour = get(envir)) +
  geom_text(aes(label=line), colour="black") +
  theme_bw() + 
  ggtitle(paste(envir)) +  
  xlab(NULL) + 
  ylab(NULL) +
  theme(axis.title.y = element_text(colour = "black", size = 24), 
        axis.text.y = element_text(size=20), 
        axis.title.x = element_text(colour = "black", size = 24), 
        axis.text.x = element_text(size=20), 
        strip.text = element_text(size=14, face="bold"),
        legend.position="none",
        legend.title = element_text(colour="black", size=20, face="bold"),
        legend.text = element_text(colour="black", size=18),
        plot.title = element_text(hjust=0.5, size=16, face = "bold"),
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank())) 

return(list(env.out, plot))

}

# specify vars to loop over
env.list <- unique(GE_comb_raw_sel$env)

plot_sel_win <- lapply(env.list, plot_sel_win_fun, df = GE_comb_raw_sel)

# specify plot-specific parameters

SxE_shoot_all <- plot_sel_all_op2[[2]] + 
  xlab(NULL) + ylab(NULL) + 
  ggtitle("All environments") +
  theme(legend.position=c(0.8,0.8))

SxE_shoot_GH <- plot_sel_win[[1]][[2]] + ggtitle("Greenhouse") +
  geom_smooth(method=lm, se=TRUE, colour="black", formula = y ~ x, linetype = 1)
SxE_shoot_p1 <- plot_sel_win[[2]][[2]] + ggtitle("Plot 1") + 
  geom_smooth(method=lm, se=FALSE, colour="black", formula = y ~ x, linetype = 2)
SxE_shoot_p2 <- plot_sel_win[[3]][[2]] + ggtitle("Plot 2") +
  geom_smooth(method=lm, se=TRUE, colour="black", formula = y ~ x, linetype = 1) +
  geom_smooth(method=lm, se=FALSE, color = "black", formula = y ~ x + I(x^2), linetype = 2)
SxE_shoot_p3 <- plot_sel_win[[4]][[2]] + ggtitle("Plot 3") +
  geom_smooth(method=lm, se=TRUE, colour="black", formula = y ~ x, linetype = 1)
SxE_shoot_p4 <- plot_sel_win[[5]][[2]] + ggtitle("Plot 4")


## put them together in cowplot

(fig_base <- plot_grid(SxE_shoot_all, SxE_shoot_GH, SxE_shoot_p1, SxE_shoot_p2,
                        SxE_shoot_p3, SxE_shoot_p4,
          ncol = 2,
          nrow = 3,
          align = "hv"))
```

![](data_analyses_SxE_files/figure-markdown_github/selection_plots-1.png)

``` r
fig.xaxis <- add_sub(fig_base, "Standardized nodules", size = 20, hjust = 0.5)

# add on the shared y axis title
fig.yaxis <- ggdraw() + draw_label("Relativized shoot biomass", size = 20, angle=90)

# put them together
fig <- plot_grid(fig.yaxis, fig.xaxis, ncol=2, rel_widths=c(0.03, 1)) 

save_plot("./figures/Fig3_SxE_shoot.pdf", fig,
          ncol = 2, ## we're saving a grid plot of 2 columns
          nrow = 3, ## and 3 rows
          # each individual subplot should have an aspect ratio of 1.3
          base_aspect_ratio = 1.3
          )
```
