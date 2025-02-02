<img src="https://imgur.com/WgXdz9r.png" />

> 💡 For better readability and detailed instructions headover to the [wiki](https://github.com/souvikinator/notion-to-md/wiki)

# Notion to Markdown

Convert notion pages, block and list of blocks to markdown (supports nesting) using **[notion-sdk-js](https://github.com/makenotion/notion-sdk-js)**

> **Note:** Before getting started, create [an integration and find the token](https://www.notion.so/my-integrations).

<a href="https://www.producthunt.com/posts/notion-to-md?utm_source=badge-featured&utm_medium=badge&utm_souce=badge-notion&#0045;to&#0045;md" target="_blank"><img src="https://api.producthunt.com/widgets/embed-image/v1/featured.svg?post_id=351669&theme=light" alt="notion&#0045;to&#0045;md - Programmatically&#0032;convert&#0032;notion&#0032;pages&#0032;to&#0032;markdown | Product Hunt" style="width: 250px; height: 54px;" width="250" height="54" /></a>

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/O5O1AFCJR)

## Todo

- [x] heading
- [x] images
- [x] quotes
- [x] links
- [x] bullets
- [x] todo
- [x] inline code
- [x] code block
- [x] strikethrough, underline, bold, italic
- [x] nested blocks
- [x] embeds, bookmarks, videos, files (converted to links)
- [x] Simple tables
- [x] toggle
- [x] divider
- [x] equation block (converted to code blocks)
- [x] convert returned markdown object to string (`toMarkdownString()`)
- [x] typescript support
- [x] add tests

## Install

```Bash
$ npm install notion-to-md
```

## Usage

> **Note:** Details on methods can be found in [API section](https://github.com/souvikinator/notion-to-md#api)

### converting markdown objects to markdown string

This is how the notion page looks for this example:

![Imgur](https://imgur.com/O6bKCmH.png)

```js
const { Client } = require("@notionhq/client");
const { NotionToMarkdown } = require("notion-to-md");
// or
// import {NotionToMarkdown} from "notion-to-md";

const notion = new Client({
  auth: "your integration token",
});

// passing notion client to the option
const n2m = new NotionToMarkdown({ notionClient: notion });

(async () => {
  const mdblocks = await n2m.pageToMarkdown("target_page_id");
  const mdString = n2m.toMarkdownString(mdblocks);

  //writing to file
  fs.writeFile("test.md", mdString, (err) => {
    console.log(err);
  });
})();
```

**Output:**

![output](https://imgur.com/XrUYrZ0.png)

### converting page to markdown object

Example notion page:

![Imgur](https://imgur.com/9iqRpBl.png)

```js
const { Client } = require("@notionhq/client");
const { NotionToMarkdown } = require("notion-to-md");

const notion = new Client({
  auth: "your integration token",
});

// passing notion client to the option
const n2m = new NotionToMarkdown({ notionClient: notion });

(async () => {
  // notice second argument, totalPage.
  const x = await n2m.pageToMarkdown("target_page_id", 2);
  console.log(x);
})();
```

**Output:**

```json
[
  {
    "parent": "# heading 1",
    "children": []
  },
  {
    "parent": "- bullet 1",
    "children": [
      {
        "parent": "- bullet 1.1",
        "children": []
      },
      {
        "parent": "- bullet 1.2",
        "children": []
      }
    ]
  },
  {
    "parent": "- bullet 2",
    "children": []
  },
  {
    "parent": "- [ ] check box 1",
    "children": [
      {
        "parent": "- [x] check box 1.2",
        "children": []
      },
      {
        "parent": "- [ ] check box 1.3",
        "children": []
      }
    ]
  },
  {
    "parent": "- [ ] checkbox 2",
    "children": []
  }
]
```

### converting list of blocks to markdown object

same notion page as before

```js
const { Client } = require("@notionhq/client");
const { NotionToMarkdown } = require("notion-to-md");

const notion = new Client({
  auth: "your integration token",
});

// passing notion client to the option
const n2m = new NotionToMarkdown({ notionClient: notion });

(async () => {
  // get all blocks in the page
  const { results } = await notion.blocks.children.list({
    block_id,
  });

  //convert to markdown
  const x = await n2m.blocksToMarkdown(results);
  console.log(x);
})();
```

**Output**: same as before

### Converting a single block to markdown string

- only takes a single notion block and returns corresponding markdown string
- nesting is ignored
- depends on @notionhq/client

```js
const { NotionToMarkdown } = require("notion-to-md");

// passing notion client to the option
const n2m = new NotionToMarkdown({ notionClient: notion });

const result = n2m.blockToMarkdown(block);
console.log(result);
```

**result**:

```
![image](https://media.giphy.com/media/Ju7l5y9osyymQ/giphy.gif)
```

## Custom Transformers

You can define your own custom transformer for a notion type, to parse and return your own string.
`setCustomTransformer(type, func)` will overload the parsing for the giving type.

```ts
const { NotionToMarkdown } = require("notion-to-md");
const n2m = new NotionToMarkdown({ notionClient: notion });
n2m.setCustomTransformer("embed", async (block) => {
  const { embed } = block as any;
  if (!embed?.url) return "";
  return `<figure>
  <iframe src="${embed?.url}"></iframe>
  <figcaption>${await n2m.blockToMarkdown(embed?.caption)}</figcaption>
</figure>`;
});
const result = n2m.blockToMarkdown(block);
// Result will now parse the `embed` type with your custom function.
```

**Note** Be aware that `setCustomTransformer` will take only the last function for the given type. You can't set two different transforms for the same type.

You can also use the default parsing by returning `false` in your custom transformer.

```ts
// ...
n2m.setCustomTransformer("embed", async (block) => {
  const { embed } = block as any;
  if (embed?.url?.includes("myspecialurl.com")) {
    return `...`; // some special rendering
  }
  return false; // use default behavior
});
const result = n2m.blockToMarkdown(block);
// Result will now only use custom parser if the embed url matches a specific url
```

## Contribution

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
Please make sure to update tests as appropriate.

## Contributers

<a href="https://github.com/souvikinator/notion-to-md/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=souvikinator/notion-to-md" />
</a>

## License

[MIT](https://choosealicense.com/licenses/mit/)
