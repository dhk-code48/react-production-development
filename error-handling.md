# ⚠️ Error Handling

## In App Errors

Utilize error boundaries in React to handle errors within specific parts of your application. Instead of having only one error boundary for the entire app, consider placing multiple error boundaries in different areas. This way, if an error occurs, it can be contained and managed locally without disrupting the entire application's functionality, ensuring a smoother user experience.

```jsx
import React from "react";
import { ErrorBoundary } from "react-error-boundary";

const DiscussionView = ({ discussionId }) => {
  // fetching discussion from backend and throw error here
  return <div>Discussion Content for {discussionId}</div>;
};

const Comments = ({ discussionId }) => {
  // fetching comments from backend and throw error here
  return <div>Comments for {discussionId}</div>;
};

const DiscussionRoute = () => {
  return (
    <div>
      <h2>Discussion {discussionId}</h2>

      <ErrorBoundary fallback={<div>Failed to load discussion. Try to refresh the page.</div>}>
        <DiscussionView discussionId={discussionId} />
      </ErrorBoundary>

      <ErrorBoundary fallback={<div>Failed to load comments. Try to refresh the page.</div>}>
        <Comments discussionId={discussionId} />
      </ErrorBoundary>
    </div>
  );
};

export default DiscussionRoute;
```

## Error Tracking

(Optional)

You should track any errors that occur in production. Although it's possible to implement your own solution, it is a better idea to use tools like [Sentry](https://sentry.io/). It will report any issue that breaks the app. You will also be able to see on which platform, browser, etc. did it occur. Make sure to upload source maps to sentry to see where in your source code did the error happen.
