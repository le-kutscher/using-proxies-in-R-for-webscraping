# Webscraping in R: using proxies

I recently did my first experiments in webscraping and I got excited about all the cool data that you can use if you know how to download it. However, I was getting worried of being blacklisted by the websites I scrape, because they see me requesting their website every 15 minutes or so. Hence, I found out that you can use proxies, that more or less request the website for you and then send it back to you (very broadly speaking). 

I was annoyed that a lot of tutorials for using proxies in R only show how this works with certain packages (e.g. rvest). I wanted a all-in-one solution, particularly because I scrape a lot of JSONs using the jsonlite-package (which doesn't have built-in proxy possibilities). So here you go, a quick and easy way to *set your proxies just once in the environment and then use it with all packages*. 

## Step 1: check your own IP (localy)
Check the configuration of your system to find your IP. 
We will not change this, because we don't want to change your IP but only redirect any request you do through a proxy. 
```{r}
sys <- system("ipconfig", intern=TRUE)    # save IP configuration
ip <- sys[grep("IPv4", sys)]              # extract IP
ip                                        # this is your IP
```

## Step 2: check your own IP (on the web)
To be sure, you can scrape a mini-website that essentially just tells you from which IP the website got your request. In other words, which IP you use when you actually go on the web. 
We use the package rvest to scrape this mini-website. 
```{r}
library(rvest)                                           # a very common library for webscraping
html_text( read_html('http://checkip.amazonaws.com/') )  # this should give you the same IP as step 1 (ignore the "\n")
```

## Step 3: tell R whicih proxy to use
First, you need to find a proxy. Just google "free proxy lists" or check this out: https://free-proxy-list.net/ . You might want to buy a proxy lateron, because these free proxies are not perfectly reliably (and you don't want your webscraping suddenly to not work on a sunday evening when you netflix & chill). 
It's important that you get both the IP of the proxy and the Port (a port is essentially one entry to the proxy server). You need to save them in the format IP:Port or URL:Port. 
Make sure that the proxies can do https (it should say that on the website from which you get the proxies), because some of your webscraping might need https and not only http. 
```{r}
proxies_list <- c("200.35.49.89:43006",   # these are 3 proxies from free-proxy-list.net
                  "80.187.140.26:8080",   # note that they have different ports
                  "154.16.63.16:8080")    # all of them can do https
                                          # FREE proxies don't always work!! So you might need to test several 
                                          # until you find one that works
proxy <- sample(proxies_list, 1)          # pick a random proxy from the list above
proxy                                     # this is the proxy we will use

Sys.setenv(https_proxy = proxy,           # write in the system environment that R should
           http_proxy = proxy)            # use this randomly picked proxy when accessing the internet
Sys.getenv(c("https_proxy", "http_proxy"))# it should be your proxy here now

```

## Step 4: check whether it works
This code should show you the same IP as the proxy. 
```{r}
html_text( read_html('http://checkip.amazonaws.com/') )    # scrape the mini-website with rvest

# it should also work with any other library, because we defined the proxy in the general R environment
library(jsonlite)
fromJSON("https://api.myip.com/")    # scraping a JSON that tells you the IP works, too
```

## Step 5: do your actual webscraping
Now you can go on and scrape the stuff you actually want to scrape. Change the proxy from time to time. 

## Step 6: switch of the usage of proxies
When you are done, tell R to use your normal IP again for everything. They are still stored in the environment and you can use the proxy again by telling R at any time. 
```{r}
Sys.setenv(no_proxy = "*")   # deactivate using proxies
#Sys.setenv(no_proxy = "")   # tell R to using the proxies again
```
