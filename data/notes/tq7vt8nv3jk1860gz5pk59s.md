
## Annotation Updates

**TL;DR** Rather than hard coding error messages at the source, provide additional attributes in the error object so we can delegate all message formatting concerns to the logger service.

1. Refactor existing type `AnnotationError` => `FieldError` (this is an unrelated type)

2. Update global state types:

    ```ts
    export const ANNOTATION_ERROR = 'AnnotationError';
    export const OBJECT_ID_NOT_FOUND = 'OBJECT_ID_NOT_FOUND';
    export const PARENT_FIELD_PATH_NOT_FOUND = 'PARENT_FIELD_PATH_NOT_FOUND';
    export const PARENT_OBJECT_ID_NOT_FOUND = 'PARENT_OBJECT_ID_NOT_FOUND';
    export const FIELD_NOT_FOUND = 'FIELD_NOT_FOUND';
    export const MULTIPLE_CONTAINER_OBJECTS = 'MULTIPLE_CONTAINER_OBJECTS';

    export type AnnotationErrorType =
        | typeof OBJECT_ID_NOT_FOUND
        | typeof PARENT_FIELD_PATH_NOT_FOUND
        | typeof PARENT_OBJECT_ID_NOT_FOUND
        | typeof FIELD_NOT_FOUND
        | typeof MULTIPLE_CONTAINER_OBJECTS;

    export interface FieldError {
        message: string;
        annotationXPath: string;
        elementXPath: string;
        attrValue: string;
        type: string;
    }

    export interface AnnotationsTreeFieldPath {
        oid: string | null;
        fp: string;
        loc: string;
        hasOnlyTextNodes?: boolean;
    }

    export interface FieldPathError {
        message: string;
        fieldPath: string;
        elementXPath: string;
        value: AnnotationsTreeFieldPath;
        type: typeof ANNOTATION_ERROR;
        kind: AnnotationErrorType;
    }

    export type AnnotationError = FieldError | FieldPathError;

    export interface State {
        fetchFieldsError: Error | null;
        fieldsErrors: FieldError[];
        clientAnnotationErrors: AnnotationError[];
        // ...
    }

    ```

3. Update `logAnnotationErrors` to provide all message formatting logic (previously encoded in `message` attribute)

4. Add a step to group errors by elementXPath prefix, display related errors as a single group.

