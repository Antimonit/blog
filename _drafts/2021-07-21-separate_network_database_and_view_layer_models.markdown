---
layout: post
title:  "Separate Network, Database and View-layer models"
date:   2021-07-18 17:50:00 +0200
tags:   programming
---

#### Separate Network, Database and View-layer models

Use separate models for updating database and to retrieve data.
Use separate models for network requests and network responses.
Use separate models for view layer.

That's also a reason why I appealed to you to distinguish network-layer models from view-layer models.
Network models should really be used only for network serialization and should not propagate to
ViewModels or even View layer!

This gives us liberty when we want to use custom types or different format than provided by BE, e.g.,
convert birthDate: String to birthDate: LocalDate,
convert (val amount: Double, val currency: String) to price: Price,
convert val title: String to val title: Text,
etc.

It is difficult to change the format of BE responses but we can create custom classes that fit
our needs better. Sharing the same model for network and views might lead to issues in the future.
You might run into a situation where BE adds a new field into the response but since you use the
model in View as well where you don't have access to this data, you are forced to mark the new
field as nullable even if you know that it will never be null when coming from BE and then you
have to handle the nullability somewhere, and it spirals on and on.
