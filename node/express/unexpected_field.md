## PROBLEM

I had an express route that was supposed to accept media files using `multer`.

```typescript
router.post<{}, IProject, CreateProjectBody>(
  '/',
  uploadSingle,
  uploadGallery,
  [
    body('title').notEmpty().withMessage('Title is required'),
    body('description').notEmpty().withMessage('Description is required'),
    body('techStack').isArray().withMessage('Tech stack must be an array'),
  ],
  createProject,
);
```
`uploadSingle` was using `upload.single('image')` and `uploadGallery` was using `upload.array('images', 5)`. 
The problem was that whenever I sent a request with both `image` and `images`, the `images` were not being uploaded.

Error:
```console
MulterError: Unexpected field
    at wrappedFileFilter (/home/akundadababalei/WebstormProjects/portfolio_cms_api/node_modules/multer/index.js:40:19)
    at Multipart.<anonymous> (/home/akundadababalei/WebstormProjects/portfolio_cms_api/node_modules/multer/lib/make-middleware.js:107:7)
    at Multipart.emit (node:events:524:28)
    at Multipart.emit (node:domain:489:12)
    at HeaderParser.cb (/home/akundadababalei/WebstormProjects/portfolio_cms_api/node_modules/busboy/lib/types/multipart.js:358:14)
    at HeaderParser.push (/home/akundadababalei/WebstormProjects/portfolio_cms_api/node_modules/busboy/lib/types/multipart.js:162:20)
    at SBMH.ssCb [as _cb] (/home/akundadababalei/WebstormProjects/portfolio_cms_api/node_modules/busboy/lib/types/multipart.js:394:37)
    at feed (/home/akundadababalei/WebstormProjects/portfolio_cms_api/node_modules/streamsearch/lib/sbmh.js:248:10)
    at SBMH.push (/home/akundadababalei/WebstormProjects/portfolio_cms_api/node_modules/streamsearch/lib/sbmh.js:104:16)
    at Multipart._write (/home/akundadababalei/WebstormProjects/portfolio_cms_api/node_modules/busboy/lib/types/multipart.js:567:19)
```
With proper error handling, I was able to see that the `images` field was at fault.

```typescript
export function errorHandler(err: Error, _req: Request, res: Response<ErrorResponse>, _next: NextFunction) {
  console.error(err.stack);
  if ((err as MulterError).field) {
    console.error('The field causing the issue is ->', (err as MulterError).field);
  }
  const statusCode = res.statusCode !== 200 ? res.statusCode : 500;
  res.status(statusCode);
  res.json({
    message: err.message,
    stack: process.env.NODE_ENV === 'production' ? '🥞' : err.stack,
  });
}
```
This error handler threw with the field saying:
```console
The field causing the issue is -> images
```

This was throwing because the first middleware for `image` would consume all
the files and see that there were file uploads for a field called `image`
as well as one for a field called `images`. However, since the `image` middleware was made to only accept
files for the `image` field, it would throw an error when it saw files for the `images` field.

## SOLUTION

The solution was to merge these two middlewares to use `upload.fields([{ name: 'images', maxCount: 5 }])` instead.


```typescript
router.post<{}, IProject, CreateProjectBody>(
  '/',
  uploadFiles,
  [
    body('title').notEmpty().withMessage('Title is required'),
    body('description').notEmpty().withMessage('Description is required'),
    body('techStack').isArray().withMessage('Tech stack must be an array'),
  ],
  createProject,
);
```
```typescript
upload.fields([
    { name: 'image', maxCount: 1 },
    { name: 'images', maxCount: 5 },
  ])
```

This way, the `uploadFiles` middleware would accept files for both `image` and `images` fields.
