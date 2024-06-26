# Creating an Answer Session

:::warning Beta feature
This is a beta feature. Please report any error on our [Slack channel](https://orama.to/slack).
:::

Once you have properly set up your index, you will be able to engage in dynamic conversations with it.

The Orama SDK exposes a very convenient function for performing highly dynamic generative AI experiences with your data through the `answerSession` API.

Let’s see what it looks like.

## Getting Started

Follow the instructions [here](/cloud/integrating-orama-cloud/javascript-sdk) to install the official JavaScript SDK. After you’ve installed it, you will be able to create your first answer session:

```ts
import { OramaClient } from '@oramacloud/client'

const orama = new OramaClient({
  endpoint: '',
  api_key: ''
})

const answerSession = orama.createAnswerSession({
  inferenceType: 'documentation',
  initialMessages: [],
  events: {
    onMessageChange:  (messages) => console.log({ messages }),
    onMessageLoading: (loading)  => console.log({ loading }),
    onAnswerAborted:  (aborted)  => console.log({ aborted }),
    onSourceChange:   (sources)  => console.log({ sources })
  }
})

await answerSession.ask({
  term: 'How do I get started with Orama?'
})
```

All the parameters above are optional, but you may need to use them to create great answer sessions for your users. Let’s see them in detail.

### `inferenceType`

`inferenceType` refers to the type of inference that runs on your data. Right now, only `documentation` is supported - therefore it’s set as a default value - but we’ll enable more inference types soon (`website`, `blog`, `ecommerce`, `generic`, and more)

### `initialMessages`

An array of initial messages for your conversational experience with Orama Cloud. By default, this parameter is an empty array, but it can be an array of objects in the following format:

```js
const initialMessages = [
  { role: 'user', content: 'What is Orama?' },
  { role: 'assistant', content: 'Orama is a next-generation answer engine' }
]
```

### `events`

The events to subscribe to. You will get notified whenever a new event gets triggered. More on this in the next section.

## Answer Session Events

Answers are a highly dynamic part of Orama. They work in an asynchronous manner and requires you to subscribe to events to catch any new change in the answer flow and react to it.

At the time of writing, Orama supports the following events:

`onMessageChange`. Runs everytime a message changes. This gets triggered with each chunk that gets streamed from the server, allowing you to re-render the messages in your page as they gets updated.

```js
const answerSession = orama.createAnswerSession({
  events: {
    onMessageChange: (messages) => {
      console.log(messages)
      // [
      //   { role: 'user', content: 'What is Orama?' },
      //   { role: 'assistant', content: 'Orama is a next-generation answer engine' }
      // ]
    },
  }
})

await answerSession.ask({
  term: 'What is Orama?'
})
```

`onMessageLoading`. Gets triggered everytime an answer request starts or ends.

```js
const answerSession = orama.createAnswerSession({
  events: {
    onMessageLoading: (loading) => {
      if (loading) {
        console.log('Still loading the messages...')
      } 
    },
  }
})

await answerSession.ask({
  term: 'What is Orama?'
})
```

`onAnswerAborted`. Gets triggered when an answer gets aborted.

```js
const answerSession = orama.createAnswerSession({
  events: {
    onAnswerAborted: (aborted) => {
      alert('The user has aborted this answer generation!')
    },
  }
})

await answerSession.ask({
  term: 'What is Orama?'
})

setTimeout(() => {
  answerSession.abortAnswer()
}, 1000)
```

`onSourceChange`. Orama Cloud will return the results of the inference process as a standard set of Orama search results. You will have access to those as soon as they gets computed:

```js
const answerSession = orama.createAnswerSession({
  events: {
    onSourceChange: (sources) => {
      console.log(`Got ${sources.count} sources in ${sources.elapsed.formatted}.`)
      console.log(`Top three sources are: ${sources.hits.map((hit) => hit.document.title).join(', ')}`)

      // Got 15 sources in 78ms.
      // Top three sources are: What is Orama, How Orama works, Why Orama is the best
    },
  }
})

await answerSession.ask({
  term: 'What is Orama?'
})
```

## Asking a Question

With the Orama JavaScript SDK, you can ask questions in two different ways:

1. using the `.ask` function
2. using the `.askStream` function

Both methods accept the [search parameters](/cloud/performing-search/full-text-search) you already use to perform search, so you can influence the inference process by filtering, sorting, and grouping the results as you prefer!

They both trigger the exact same events, but the only difference is how they return the answer.

The `.ask` function will return the entire answer once it has been entirely streamed from the server, therefore, you can use this method as follows:

```jsx
const answer = await answerSession.ask({
  term: 'What is Orama?'
})

console.log(answer)
// Orama is a next-generation answer engine
```

The `.askStream` method returns an async iterator that yields chunks of the answer as they gets streamed back from the server. Therefore, this is how you would implement it:

```jsx
const answer = await answerSession.askStream({
  term: 'What is Orama?'
})

for await (const msg of answer) {
	console.log(msg)
}

// Orama
//  is a
// next-gener
// ation answer
//  engine
```

## Aborting Answers Generation

In any moment in time (when a user cancels an answer request, for example) you can abort an answer by using the `.abortAnswer` function:

```jsx
answerSession.abortAnswer()
```

This will trigger the `onAnswerAborted` event, which will simply return a `true` in its callback function:

```jsx
const answerSession = orama.createAnswerSession({
  events: {
    onAnswerAborted: (aborted) => {
	    alert('The user aborted the answer session!')
    }
  }
})
```

## Getting all the Messages

At any point in time, you can retrieve all the messages by calling the `.getMessages` function:

```js
const messages = answerSession.getMessages()

console.log(messages)

// [
//   { role: 'user', content: 'What is Orama?' },
//   { role: 'assistant', content: 'Orama is a next-generation answer engine' }
// ]
```

## Clearing the Session

You can clear the answer session by using the `clearSession` method. This will reset the messages to an empty array:

```js
answerSession.clearSession()
```