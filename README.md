mongoose-uniqueslugs plugin
===========================

This plugin guarantees unique slugs for model objects. Specifically, this plugin can extend any Mongoose model that 
has a unique index on a 'slug' field so that if a unique index error occurs on 'slug', a random digit is added to 
the slug and the save operation is retried until it works. This is concurrency-safe even if you have lots of inserts 
going on from multiple machines, etc. 

Here's how to use it. It differs from other plugins because we must modify the model object to add a save() 
wrapper that gets first crack at errors.

1. Make your schema the usual way (leave out the slug field, we'll add that for you)

2. Require the unique slug plugin

    var mongooseUniqueSlugs = require('mongooseUniqueSlugs');

3. Enhance the schema

    mongooseUniqueSlugs.enhanceSchema(mediaItemSchema);

4. Get your model

    var MediaItem = mongoose.model('MediaItem', mediaItemSchema);

5. Enhance your model. (This is necessary in order to retry save() operations correctly)

    mongooseUniqueSlugs.enhanceModel(MediaItem);

ALSO: note that it is also very important to make sure your indexes have really 
been applied before you try to do things like inserts! Otherwise you may not get 
your unique index error until it is too late.
 
    MediaItem.on('index', function()
    {
      nowICanSafelyInsertThings();
    });

All save() operations now automatically get upgraded to use "safe: true" in order to make this check for slug uniqueness possible.

Options
=======

You can pass options as a second parameter to enhanceSchema(). The available options are:

source: if defined, this is the property to be converted to a slug. If not defined, 'title' is assumed.

disallow: if defined, this is a regular expression object that matches characters that should be REMOVED from the slug. If not defined, the following regular expression is used, replacing everythign except letters and numbers with '-' (unless that is overridden also):

    /[^\w\d]+/g;

substitute: if defined, this is the character used to replace characters that are not permitted in slugs. Note that runs of more than one substitute character are folded to just one, and any substitute characters at the beginning and end are automatically removed. If not defined, '-' is assumed.

addSlugManually: if true, the plugin will NOT add a unique slug field to your schema automatically. Use this option when you prefer to add the field yourself, for instance so that you can define a compound index on a second field (you must still define some kind of unique index). If not specified, a 'slug' field is added automatically with a unique index on that field only.
