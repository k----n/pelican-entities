# pelican-entities

A new generator for [Pelican](http://pelican.readthedocs.org/en/latest/) that
replaces the default Page and Article generators, allowing the definition of
arbitrary content types, aka, entity types (e.g: projects, events) and
associated indices/direct templates.

## Install

To install the library, you can use
[pip](http://www.pip-installer.org/en/latest/).

```bash
$ pip install pelican-minify
```


## Usage

1. Update `pelicanconf.py`:
    1. Add `pelican-entities` to `PLUGINS`.
            
            PLUGINS = ['pelican-entities', ...]

    2. Disable default page and article generators:
            
            PAGE_PATHS = []
            ARTICLE_PATHS = []
            DIRECT_TEMPLATES = []
            PAGINATED_DIRECT_TEMPLATES = []

    3. Specify entity types to use in your site and their settings:

            ENTITY_TYPES = {
                <type1_name>: {
                    PATHS: [<type1_path1>, <type1_path2>, ...],
                    EXCLUDES: [...],
                    <type1_name>_URL: "...",
                    <type1_name>_SAVE_AS: "...",
                    ...
                },
                <type2_name>: {
                    ...
                }
            }

2. Update theme to use new variables.
3. Watch out for incompatible plugins (those that rely on the pages and 
   article generator signals or on the existence of variables defined by
   them).


## Settings

Settings defined at the ENTITY_TYPES level take precedence over global
ones.

### Changes from default
* Settings for each entity type are defined as the value of the corresponding
  key in the `ENTITY_TYPES` dictionary.
* `PAGE_PATHS` and `ARTICLE_PATHS` are replaced by `PATHS`.
* `PAGE_EXCLUDES` and `ARTICLE_EXCLUDES` are replaced by `EXCLUDES`.
* `PAGE_URL`, `ARTICLE_URL`, `PAGE_SAVE_AS` and `ARTICLE_SAVE_AS` replaced by
  generic `<entity_name>_URL` and `<entity_name>_SAVE_AS`.

### New settings
* `DEFAULT_TEMPLATE`: Template to use by default when generating pages for
  that entity type.
* `MANDATORY_PROPERTIES`: List of properties that has to be defined for an
  entity to be considered valid (by default, just date).
* `SORT_ATTRIBUTES`: List of properties used to sort entities. Date is 
  always used as the last attribute.
* `ARCHIVE_TEMPLATE`: Template used for archive pages.
* `CATEGORY_TEMPLATE`: Template used for category pages.
* `TAG_TEMPLATE`: Template used for tag pages.
* `AUTHOR_TEMPLATE`: Template used for author pages.

## Themes

### New available variables

* Global:
    * `url`: The url of the current page.
    * `entity_type`: Type of the entity associated with this page.
* Entity page:
    * `entity`: Contains the object describing an entity (replaces `article`
       or `page`).
* Direct templates:
    * `direct`: Variable always equal to True when rendering a direct template.
* Tag, category, author pages:
    * `entities`: Replaces `articles`.
    * `all_entitites`: Replaces `all_articles`.
* Draft pages:
    * `entity`: Replaces `article`.
    * `all_entities`: Replaces `all_articles`.
* Paginated pages (direct templates or tag, category, author pages):
    * `entities_paginator`: Replaces `articles_paginator`.
    * `entities_page`: Replaces `articles_page`.
    * `entities_previous_page`: Replaces `articles_previous_page`.
    * `entities_next_page`: Replaces `articles_next_page`.

### Deleted variables
* Entity page:
    * `category`: Access through `entity.category`.
* Direct templates:
    * `dates`: If you want to iterate in the opposite order do it explicitly.

## Example configuration

This is the configuration I'm using on my site:

``` python
ENTITY_TYPES = {
    "Page": {
        "PATHS": ["."],
        "EXCLUDES": ["blog", "projects"],
        "PAGE_URL": "{slug}",
        "PAGE_SAVE_AS": "{slug}/index.html",
        "PATH_METADATA": r"(?P<slug>[^/]+)/.*",
        "DIRECT_TEMPLATES": ["search"],
        "SEARCH_SAVE_AS": "search/index.html"
    },
    "Article": {
        "PATHS": ["blog"],
        "ARTICLE_URL": "blog/{category}/{slug}/",
        "ARTICLE_SAVE_AS": "blog/{category}/{slug}/index.html",
        "PATH_METADATA": r".*/(?P<category>[^/]+)/(?P<date>\d{4}/\d{2}/\d{2})/(?P<slug>[^/]+)/.*",
        "DIRECT_TEMPLATES": ["blog"],
        "PAGINATED_DIRECT_TEMPLATES": ["blog"],
        "BLOG_SAVE_AS": "blog/index.html",
        "CATEGORY_TEMPLATE": "blog_category",
        "CATEGORY_URL": "blog/{slug}/",
        "CATEGORY_SAVE_AS": os.path.join("blog", "{slug}", "index.html"),
        "FEED_ATOM": os.path.join("blog", "feeds", "atom.xml"),
        "CATEGORY_FEED_ATOM": os.path.join("blog", "feeds", "%s.atom.xml")
    },
    "Project": {
        "PATHS": ["projects"],
        "SORT_ATTRIBUTES": ["project_start"],
        "PROJECT_URL": "projects/{category}/{slug}/",
        "PROJECT_SAVE_AS": "projects/{category}/{slug}/index.html",
        "PATH_METADATA": r".*/(?P<category>[^/]+)/(?P<slug>[^/]+)/.*",
        "DIRECT_TEMPLATES": ["projects"],
        "PAGINATED_DIRECT_TEMPLATES": ["projects"],
        "PROJECTS_SAVE_AS": "projects/index.html",
        "CATEGORY_TEMPLATE": "project_category",
        "CATEGORY_URL": 'projects/{slug}/',
        "CATEGORY_SAVE_AS": os.path.join('projects', '{slug}', 'index.html'),
        "FEED_ATOM": os.path.join("projects", "feeds", "atom.xml"),
        "CATEGORY_FEED_ATOM": os.path.join("projects", "feeds", "%s.atom.xml")
    }
}
```

For a working example check [my site](www.alexjf.net) and [my site's source
code](https://github.com/AlexJF/alexjf.net).

## Extending

### Available signals

* `entity_generator_init`: Initialization of the parent generator. This
  generator is responsible for creating the generators for each entity type.
* `entity_generator_finalized`: End of context generation by the parent
  generator.
* `entity_writer_finalized`: End of output generation by the parent generator.

* `entity_subgenerator_*`: Signals for the generator of a particular entity
  type. These are the same signals used by the article generator.