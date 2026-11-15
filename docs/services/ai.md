---
title: "AI"
icon: "✨"
created: 2026-11-15
updated: 2026-11-15
---

# AI

`Sandbox.Services.Ai` lets your game ask questions, describe images, generate structured data, and hold conversations that can call your game code. Requests use the current player's authenticated s&box session and AI credits. You don't need to supply a provider API key.

## Asking a question

Create a configuration, build a query, then run it. The configuration describes how the AI should behave; the query contains what you want it to answer.

```csharp
using Sandbox.Services;

var config = new Ai.Config
{
    Instructions = "You are a harbour keeper. Give short hints without revealing puzzle solutions."
};

var response = await config.CreateQuery( "How can I find the missing sailor?" ).Run();

if ( response.IsSuccess && string.IsNullOrEmpty( response.Refusal ) )
{
    Log.Info( response.Text );
}
else
{
    Log.Warning( response.Refusal ?? $"The reply ended with {response.FinishReason}." );
}
```

The examples below use `using Sandbox.Services;`. Run the `await` examples from an asynchronous method on the game context.

Each query is independent. Supply any game state it needs in its input, or use a [chat](#conversations) to keep the previous turns. Creating a query or chat copies the configuration, so changing `Instructions` or registering more tools afterwards won't change an existing one.

`Run()` returns an `Ai.Response`. Check `IsSuccess` and `Refusal` before using `Text`: a response can contain partial text without completing successfully. `FinishReason` explains why it stopped, such as `Stop`, `Length`, or `ContentFilter`. Network and service failures throw an [exception](#handling-errors).

## Structured responses

Use `Run<T>()` when you want data instead of prose. The API builds a JSON schema from your type and deserializes the reply for you. XML summaries describe the type and its properties to the AI.

```csharp
/// <summary>
/// A short hint to display in the quest journal.
/// </summary>
public sealed class QuestHint
{
    /// <summary>
    /// A short title for the hint.
    /// </summary>
    public string Title { get; set; }

    /// <summary>
    /// A clue that points the player towards the next step without giving away the solution.
    /// </summary>
    public string Clue { get; set; }
}
```

```csharp
var response = await config.CreateQuery(
    "The player has found a torn map near the docks. Suggest where to look next."
).Run<QuestHint>();

if ( response.HasData )
{
    Log.Info( $"{response.Data.Title}: {response.Data.Clue}" );
}
else
{
    Log.Warning( response.Refusal ?? $"No structured answer: {response.FinishReason}." );
}
```

Always check `HasData` before reading `Data`. It is false for refusals, incomplete replies, and invalid JSON. The original text and credit usage are still available on the response.

Queries without custom tools use the backend's strict output schema. Queries with custom tools describe the output schema in their instructions and parse the final reply, so they can still return text that doesn't match your type. Validate values against your game's rules before using them.

## Images

Add images to a query with `AddImage`. You can supply an HTTP(S) URL, a base64 image data URL, a `Bitmap`, or a 2D `Texture`.

```csharp
var query = config.CreateQuery( "Describe the objects in this image." )
    .AddImage( "https://example.com/puzzle.png" );

var response = await query.Run();
```

Replace the example URL with an image the provider can access. URLs are forwarded to the provider; the game doesn't download them first. You can omit the text when you supply an image, and use multiple `AddImage` calls for multiple images.

For a bitmap or texture you already have:

```csharp
var query = config.CreateQuery( "Read the writing on this sign." );
query.AddImage( bitmap, lossless: true );
```

| Input | Behaviour |
| --- | --- |
| `Bitmap` | Copied and encoded during `AddImage`, so you can dispose the original afterwards. |
| `Texture` | Read from the GPU when the query is submitted. Keep it alive until submission finishes. The encoded image is cached for reuse and retries. |
| Base64 data URL | Accepts JPEG, PNG, WebP, or GIF, with a maximum of 3 MiB before base64 encoding. |

Bitmaps and textures are resized to fit within 2048 pixels on their longest side. They use JPEG by default; `lossless: true` uses PNG, which is useful for text, diagrams, and transparency. The encoded image must fit within 3 MiB. The combined image URL/data-URL payload for a submission must fit within 16 MiB.

## Conversations

Keep an `Ai.Chat` when later questions should remember earlier replies.

```csharp
var chat = config.CreateChat();

var first = await chat.Send( "My name is Alex. I'm looking for the missing sailor." );
var second = await chat.Send( "What do you know about him?" );
```

Creating a chat doesn't make a server request. `Send` submits the conversation and returns the final assistant response, automatically running any tools it requests. Check each response as you would a query response.

`chat.Messages` contains acknowledged user messages and final assistant replies. `chat.LastResponse` is the latest final reply. Keep the chat instance for as long as you need that conversation; creating another chat starts with no history.

To send images, pass a query as the message:

```csharp
var message = config.CreateQuery( "Is this the place you mentioned?" )
    .AddImage( "https://example.com/docks.png" );

var response = await chat.Send( message );
```

The chat uses the query's input and images, but keeps its own instructions and tools. Outgoing conversation history retains up to 16 MiB of image content, replacing older images with text placeholders as needed.

### Retrying and resetting

Only one operation can run on a chat at a time. Await `Send` or `Retry` before starting another operation.

If a request fails or is cancelled after the turn starts, the chat retains that turn. Use `Retry()` to continue it instead of sending the same user message again:

```csharp
var response = await chat.Retry();
```

`Retry` requires a pending turn. Input validation or image preparation can fail before a turn starts; fix that input and call `Send` again instead.

Completed local tool calls aren't rerun during a retry. A tool cancelled while executing is recorded as a tool error, since it may already have changed the game. Cancellation doesn't undo game actions or guarantee that server processing stopped. If a response was lost in transit, retrying may generate and charge for it again.

Use `chat.Reset()` to abandon a pending turn or start over. It clears the history, last response, and chat usage total, and creates a new chat ID while keeping the same configuration. It doesn't refund credits or undo tool actions.

## Giving the AI tools

Tools let the AI read game state or request actions through code you provide. Derive from `Ai.BaseTool<TResult>`, add public input properties, and implement `Run`. XML summaries describe the tool, inputs, and result to the AI.

This tool reads a hint from a dictionary owned by the game:

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Sandbox.Services;

/// <summary>
/// Reads the game's hint for a quest using its internal ID.
/// </summary>
public sealed class GetQuestHint : Ai.BaseTool<string>
{
    /// <summary>
    /// The quest's internal ID, such as missing_sailor.
    /// </summary>
    public string QuestId { get; set; }

    /// <summary>
    /// Looks up the requested quest without changing the player's progress.
    /// </summary>
    public override Task<string> Run( CancellationToken cancellationToken )
    {
        cancellationToken.ThrowIfCancellationRequested();

        var hints = (IReadOnlyDictionary<string, string>)Context;

        if ( string.IsNullOrWhiteSpace( QuestId ) || !hints.TryGetValue( QuestId, out var hint ) )
        {
            return Task.FromResult( "No hint is available for that quest." );
        }

        return Task.FromResult( hint );
    }
}
```

Register it before creating the chat or query:

```csharp
var hints = new Dictionary<string, string>
{
    ["missing_sailor"] = "Ask the lighthouse keeper about the last boat."
};

var config = new Ai.Config
{
    Instructions = "Help the player with quests. Use GetQuestHint to look up hints.",
    Context = hints
}.AddTool<GetQuestHint>();

var chat = config.CreateChat();
var response = await chat.Send( "Give me a hint for missing_sailor." );
```

`Context` is a game-owned object passed to each tool. It isn't sent to the AI; the tool's returned result is. The configuration keeps the same object reference, so tools can read live state from it.

Each invocation gets a new tool instance populated with the AI's arguments. Use unique class names for tools, and validate their inputs and game permissions inside `Run`. A model requesting an action doesn't grant authority to perform it. Tool exceptions are returned to the AI as tool errors, allowing it to respond or try another approach.

Tools normally return JSON-serializable data. They can also return a `Bitmap` or `Texture` for a single image, or an `Ai.Query` containing text and images. A returned query is treated as tool output; its configuration is ignored and it doesn't start another AI request. Keep returned bitmaps and textures alive until the chat has prepared them.

### Limiting tool calls

`Config.MaxToolRounds` defaults to 30. It limits batches of tool calls per turn, including retries, rather than the number of individual calls. Reaching the limit throws `InvalidOperationException`.

For a chat, you can raise `chat.MaxToolRounds` before calling `Retry`, or use `Reset` to abandon the turn. For a tool-enabled query, the limit is terminal: create a new query with a higher configured limit if needed. For other interruptions, call `Run` again on the same query to resume its retained tool turn without repeating completed local tools.

## Credits

AI usage is charged to the current player's account. Read `Ai.Credits` for the last known balance:

```csharp
if ( Ai.Credits is { } credits )
{
    Log.Info( $"AI credits: {credits.Available}. Renews at {credits.RenewsAt:u}." );
}
```

The balance is nullable: `null` means it isn't available yet, not that the player has no credits. Login and AI responses refresh it automatically, and reading it doesn't make a request. Use `await Ai.RefreshCredits()` when you need an explicit refresh, such as after a credit grant. Avoid polling it every frame.

`Available` can be negative after an overrun. `RenewsAt` is the next monthly allowance renewal, at the start of the next calendar month in UTC.

`response.CreditsSpent` is the cost of that response. For a chat or tool-enabled query, it describes the final reply, so use `response.TotalCreditsSpent` or `chat.TotalCreditsSpent` to include the preceding tool rounds. Usage values can be `null` when accounting is unavailable; don't display that as zero.

## Handling errors

Network and service failures throw `Ai.RequestException`, with a `Code`, optional HTTP `StatusCode`, and message. Errors may also carry a credit balance and renewal date.

```csharp
try
{
    var response = await config.CreateQuery( "Give me a short greeting." ).Run();

    if ( response.IsSuccess && string.IsNullOrEmpty( response.Refusal ) )
    {
        Log.Info( response.Text );
    }
    else
    {
        Log.Warning( response.Refusal ?? $"The reply ended with {response.FinishReason}." );
    }
}
catch ( Ai.RequestException exception )
{
    if ( exception.Code == "insufficient_credits" )
    {
        Log.Warning( $"No AI credits remaining. Renews at {exception.RenewsAt}." );
    }
    else
    {
        Log.Warning( $"AI request failed ({exception.Code}): {exception.Message}" );
    }
}
```

Use `Code` rather than matching error message text. Examples include `unauthorized` for a missing or invalid session, `insufficient_credits`, `credits_unavailable`, and `network_error`. A model refusal is returned in the response instead of being thrown as a request error.

`Run`, `Send`, `Retry`, and `RefreshCredits` accept a `CancellationToken`. For the string overload of `Send`, use the named argument: `await chat.Send( "Hello", cancellationToken: token )`. Cancellation throws `OperationCanceledException`; input validation and concurrent chat operations can throw argument or invalid-operation exceptions. Handle these separately from service errors.
