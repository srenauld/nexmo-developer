---
title: Javascript
language: javascript
menu_weight: 1
---

```javascript
/** 
 *  Debounced iterator to process a paginated dataset
 *  and return a list of conversations
 */ 
function processAndGetNextConversationPage(pageRecord, { current, total }) {
  return new Promise((resolve, reject) => {
    const results = Array.from(pageRecord.items.values()) || [];
    const newCount = current + results.length;
    if (current >= total) {
      return resolve(results);
    }
    if (!pageRecord.hasNext()) {
      return resolve(results);
    }
    return pageRecord.getNext()
      .then((newRecord) => processAndGetNextConversationPage(
        newRecord,
        { current: newCount, total },
      ))
      .then((newResults) => {
        for (let i = 0; i < newResults.length; i += 1) {
          results.push(newResults[i]);
        }
        return resolve(results);
      })
      .catch(reject);
  });
}

function listConversations(app, maxTotalRecords) {
  return new Promise((resolve, reject) => app.getConversations({
    order: 'asc',
    page_size: 1,
  }).then((conversationPage) => processAndGetNextConversationPage(
    conversationPage,
    {
      current: 0,
      total: maxTotalRecords,
    },
  )).then(resolve).catch(reject));
}

const app = client.login(token)
.then(() => {
  listConversations(app, 50)
    .then(console.log)
    .catch(console.log);
});
```
