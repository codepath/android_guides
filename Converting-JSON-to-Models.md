## Overview

This guide describes the process of converting JSON data retrieved from a network request and converting that data to simple Model objects. This approach will be compatible with nearly any basic data-driven application and fits well with any ORM solution for persistence such as [ActiveAndroid](http://www.activeandroid.com/) or [SugarORM](http://satyan.github.io/sugar/) that may be introduced.

For this guide, we will be using the [Yelp API](http://www.yelp.com/developers/documentation/v2/search_api#searchGP) as the example. The goal of this guide is to **perform a Yelp API Search** and then **process the results into Java objects** which we can then use to populate information within our application.

The model in this case is **Business** and for our application, let's suppose we just need the _name_, _phone_, and _image_ of the business.

### Fetching JSON Results

