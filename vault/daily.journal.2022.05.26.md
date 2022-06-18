---
id: q7aj38uv8dnumqrpxn10vk6
title: '2022-05-26'
desc: ''
updated: 1653647969036
created: 1653627313511
traitIds:
  - journalNote
---


## Minimal Examples

The goal of this experiment was to produce 3 minimal examples using only the online documentation as a guide. These examples highlight the differences between CMS providers by providing a common model and runtime implementation shared across the examples.

Tested providers:

- git
- contentful
- custom

conclusion: The documentation was easy to follow, seemed complete, and there were hardly any issues writing each implementation to run locally on the first try. However, **in the end, not a single one was editable in Stackbit.**

### Content Model

We will keep the model as small as possible to reduce the potential source of errors.

```ts

/**
 * Loaded into commonProps as `site`
 */
interface SiteConfig {
  title: string;
}

/**
 * A simple page model.
 */
interface Page
  title: string;
  body: string;
  slug: string;
}
```

### Component Implementation

We should be able to rely on sourcebit to abstract away differences in model sources for greater code reuse in the UI.

**Example**

```tsx
import Head from "next/head";
import { sourcebitDataClient } from "sourcebit-target-next";
import { toObjectId, toFieldPath } from "@stackbit/annotations";
import { withRemoteDataUpdates } from "sourcebit-target-next/with-remote-data-updates";
import styles from "../styles/Home.module.css";

export function DynamicPage({ site, page }) {
  return (
    <div className={styles.container}>
      <Head>
        <title>{site.title}</title>
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main className={styles.main} {...toObjectId(page.__metadata.id)}>
        <h1 className={styles.title} {...toFieldPath("title")}>
          {page.title}
        </h1>
        <p className={styles.description} {...toFieldPath("body")}>
          {page.body}
        </p>
      </main>
    </div>
  );
}

export async function getStaticProps({ params }) {
  const pagePath =
    typeof params?.slug === "string"
      ? params?.slug
      : "/" + (params?.slug || []).join("/");
  const props = await sourcebitDataClient.getStaticPropsForPageAtPath(pagePath);
  return { props };
}

export async function getStaticPaths() {
  const paths = await sourcebitDataClient.getStaticPaths();
  return {
    paths,
    fallback: false,
  };
}

export default withRemoteDataUpdates(DynamicPage);
```

In theory, there is no reason why this page couldn't be reused for all 3 examples, once the entries are correctly bucketed by `sourcebit-target-nextjs`. One should only need to modify `sourcebit.js` and `stackbit.yaml` to swap out a model source.

### Shared Utility Code

Ideally, we should also be able to rely on a some common  __metadata fields that allows us to reuse common utility code as well.

**Example**

```ts
{
    module: require('sourcebit-target-next'),
    options: {
        pages: function (objects, utils) {
            return objects.reduce((pages, page) => {
                if (page.__metadata.modelName === 'Page') {
                    return pages.concat({
                        path: '{slug}',
                        page
                    });
                }

                return pages;
            }, []);
        },
        commonProps: function (objects, utils) {
            return {
                site: objects.find((object) => object.__metadata.modelName === 'SiteConfig')
            };
        }
    }
}
```

note: This functionality is not yet part of sourcebit, and may not even be one of its defined goals.

### The examples

#### [stackbit-examples/minimal-filesystem](https://github.com/stackbit-themes/stackbit-examples/tree/minimal-examples/minimal-filesystem)

Plugins: `sourcebit-source-filesystem`, `sourcebit-target-next`

##### Notes

1. By default the `sourcebit-source-filesystem` plugin does not respect a user's domain model, and will apply opinionated transformations to it instead. This makes any code reuse impossible without writing custom plugins to override this behavior:
  
2. All fields are placed in a `frontmatter` object, rather then kept at the top level of the entry.

3. The resolved model name is not kept in metadata like other api based plugins

4. An additional user defined method is needed to derive the slug from the file path, which wasn't very clear at first. 

##### Validator results

```
➜  minimal-filesystem git:(minimal-examples) ✗ yarn stackbit validate
yarn run v1.22.18
$ stackbit validate
loading and validating Stackbit configuration from: stackbit-examples/minimal-filesystem
  loaded 2 models:
    ✔ Page
    ✔ SiteConfig
  ✔ configuration is valid
loading and validating content from: stackbit-examples/minimal-filesystem
  loaded 2 files in total (2 matched, 0 unmatched)
  2 files matched to models:
    SiteConfig: 1 files:
      content/data/config.json
    Page: 1 files:
      content/pages/index.md
  ✔ content files are valid
✔ validation passed
Done in 1.48s.
```


#### [stackbit-examples/minimal-contentful](https://github.com/stackbit-themes/stackbit-examples/tree/minimal-examples/minimal-contentful)

Plugins: `sourcebit-source-contentful`, `sourcebit-target-next`


##### Validation results

```
➜  minimal-contentful git:(minimal-examples) yarn validate
yarn run v1.22.18
$ stackbit validate
loading and validating Stackbit configuration from: stackbit-examples/minimal-contentful
  loaded 3 models:
    ✔ Page
    ✔ SiteConfig
  ✔ configuration is valid
Done in 1.41s.
```

#### [stackbit-examples/minimal-sourcebit](https://github.com/stackbit-themes/stackbit-examples/tree/minimal-examples/minimal-sourcebit)

Plugins: `sourcebit-source-contentful`, `sourcebit-target-next`, `sourcebit-sample-plugin` (local)

##### Notes

1. An additional field needs to be added to page models to use as a slug.
2. Contains models merged in from file system and custom plugin
3. Uses yarn workspaces to define plugin package
4. It was unclear what to use as the `cmsName` in `stackbit.yaml`.

##### Validator results

```
➜  minimal-sourcebit git:(minimal-examples) ✗ yarn stackbit validate 
yarn run v1.22.18
$ stackbit validate
loading and validating Stackbit configuration from: stackbit-examples/minimal-sourcebit
  loaded 2 models:
    ✔ Page
    ✔ SiteConfig
  ✔ configuration is valid
loading and validating content from: stackbit-examples/minimal-sourcebit
  loaded 1 files in total (1 matched, 0 unmatched)
  1 files matched to models:
    SiteConfig: 1 files:
      content/data/config.json
  ✔ content files are valid
✔ validation passed
Done in 1.42s.
```

## Inconsistencies w/ documentation

Overall the documentation seemed very complete and consistent with what's in our themes, with only a few minor exceptions.


### stackbit.yaml

There appears to be a difference in how models are defined in git-based and api-based CMS. The documented method did not appear to work for the latter.

Documented: (does not work with API-based CMS)

```yml
contentModels:
  Page:
    isPage: true
    urlPath: "/{slug}"
```

Undocumented: (works in cms)

```yml
models:
    Page:
        type: page
        urlPath: '/{slug}'
```

### HOC for live preview

The documentation refers to a different HOC to use when exporting page components than what is used in the starter template and themes. It was unclear which is the correct version to use.

`sourcebit-target-next/with-remote-data-updates`

vs

`sourcebit-target-next/hot-content-reload`

There was also a small error in the docs which actually had already had a PR to fix it submitted by a community member. I went ahead and merged it. 
