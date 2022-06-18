---
id: b3w46cf4yjy2d8iitxoymmb
title: '2022-06-02'
desc: ''
updated: 1654212754962
created: 1654212375843
traitIds:
  - journalNote
---

stackbit/packages/dev-common/src/runner/editor.js

## 1. TODO move out of common package

L370

```ts
async getObjectsWithAnnotations({ annotationTree = null, clientAnnotationErrors = [], locale, reportAllErrors = false }) {
    const fieldData = this.cms.getLocalizedFieldData(locale);
    const { objects, pathMap, errors } = resolveObjectsWithAnnotationTree(annotationTree, fieldData, { resolveAllReferences: true });
    this.populateReferenceLabels(objects, fieldData);
    //TODO move out of common package
    if (!reportAllErrors) {
      const allErrors = [...errors, ...clientAnnotationErrors];
      const displayErrors = allErrors.slice(0, 5);
      _.forEach(displayErrors, error => {
          const errorProps = _.omit(error, ['type', 'message']);
          this.userLogger.error(
              `${error.message} ${_.map(errorProps, (val, key) => `${key}:'${val && val.toString ? val.toString() : val}'`).join(', ')}`
          );
      });
      const diffErrors = allErrors.length - displayErrors.length;
      if (diffErrors > 0) {
          this.userLogger.error(
              `${displayErrors.length} Annotation errors shown (out of ${allErrors.length}). Use \`stackbit dev\` locally to view all.`
          );
      }
    }
    return { objects, pathMap, errors };
}
```

## 2. ssg-server (POST /_objectsWithAnnotations)


```ts
server.post('/_objectsWithAnnotations', [bodyParser.json({ limit: '500kb' })], (req, res, next) => {
      logger.debug('[ssg-server] get objects with annotations');
      const { annotationTree, clientAnnotationErrors, locale } = req.body;
      return runner.getObjectsWithAnnotations({ annotationTree, clientAnnotationErrors, locale }).then(result => {
          logger.debug('[ssg-server] done getting objects with annotations');
          res.json(result);
      }).catch(err => {
          logger.error('[ssg-server] error getting fieldData with annotations from runner', {error: err.message || err});
          if (_.has(err, 'stack')) {
              logger.error('[ssg-server] error getting fieldData with annotations from runner, error.stack: ' + err.stack);
          }
          res.status(500).send('Error');
          next(err);
      });
  });
```
