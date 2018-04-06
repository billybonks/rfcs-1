- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

Refactor [#jsHintAddon](https://github.com/ember-cli/ember-cli/blob/master/lib/models/addon.js#L1188), to follow the way the pattern that assets are compiled with [#treeFor](https://github.com/ember-cli/ember-cli/blob/master/lib/models/addon.js#L524)

# Motivation

As Ember cli paths and directories grow change, we need to manage [trees](https://github.com/ember-cli/ember-cli/blob/master/lib/broccoli/ember-addon.js#L35) but we also need to manage completely independently [#jsHintAddon](https://github.com/ember-cli/ember-cli/blob/master/lib/models/addon.js#L1188) This causes bugs across all linters where trees are not passed or they don't work the way the code flows when compiling addons.

# Detailed design

[#treeFor](https://github.com/ember-cli/ember-cli/blob/master/lib/models/addon.js#L524), gets called for every `type` of tree from the [_addonTreesFor](https://github.com/ember-cli/ember-cli/blob/master/lib/broccoli/ember-app.js#L657) the types are
```
src
test-support
addon-test-support
addon
styles
templates
app
vendor
```

and trees have been already built for each of these types and passed as paramter to [#treeFor](https://github.com/ember-cli/ember-cli/blob/master/lib/models/addon.js#L524).

How we lint at the moment, is if the type is `app` we call [#jsHintAddon](https://github.com/ember-cli/ember-cli/blob/master/lib/models/addon.js#L1188), and we start to build random trees as we see fit.

- firstly this increase the amount of code that needs to be managed as ember evolves
- secondly we are doing duplicate work decreasing performance of ember cli.

The goal of this rfc is to decrease complexity of ember-cli and to increase stability of addons, so insead of

```
if (this.isDevelopingAddon() && this.hintingEnabled() && type == "app") {
  trees.push(this.jshintAddonTree());
}
```

we should just

```
if (this.isDevelopingAddon() && this.hintingEnabled() && type == "app") {
  this._eachProjectAddonInvoke('lintTree', [type, tree]);
}

```
This change should not break existing liners because all the linters i have looked at use `broccoli-filter` to filter files that they lint. since there aren't many linting addons we can open a pr with a custom branch of ember-cli to see if it breaks their ci

https://emberobserver.com/?query=lint

`jsHintAddon` method should be deprecated.

we would have to update ember-cli API

# Drawbacks

the `app` lint trees still need to managed manually with [lintTestTrees](https://github.com/ember-cli/ember-cli/blob/master/lib/broccoli/ember-app.js#L1462)

# Alternatives

I tried using to find the trees within the [#jsHintAddon](https://github.com/ember-cli/ember-cli/blob/master/lib/models/addon.js#L1188) function instead of building new ones

# Unresolved questions

How does this affect ember-engines
