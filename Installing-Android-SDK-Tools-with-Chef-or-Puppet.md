If you intend to incorporate Android builds into your continuous integration setup, you may want to use [Chef](https://learn.chef.io/) or Puppet to help automate this process.  These configuration management tools help automate the process of managing machine configurations in Amazon, Rackspace, or Linode.   

## Chef recipes

For Chef, you can download the chef-android-sdk recipe.  It has a few dependencies, including Ark and 7-zip:

```
wget https://github.com/gildegoma/chef-android-sdk/archive/master.zip
wget https://github.com/burtlo/ark/archive/master.zip
wget https://github.com/sneal/7-zip/archive/master.zip

```

Assuming you install chef-android-sdk, this recipe should install the latest Android SDK tools and setup the environment variables in `/etc/profile.d/android-sdk.sh`.