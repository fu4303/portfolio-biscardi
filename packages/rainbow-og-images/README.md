# gatsby-plugin-printer

## Node API

This is a declarative API that lets the library control batching, caching, etc. Creating `Printer` nodes via this API doesn't give you a `fileName` back, so you would have to specify one.

```js
import { createPrinterNode } from "gatsby-plugin-printer";

exports.onCreateNode = ({ actions, node }) => {
  if (node.internal.type === "Mdx") {
    // createPrinterNode creates an object that can be passed in
    // to `createNode`
    const printerNode = createPrinterNode({
      id: node.id,
      // fileName is something you can use in opengraph images, etc
      fileName: slugify(node.title),
      // renderDir is relative to `public` by default
      renderDir: "blog-post-images",
      // data gets passed directly to your react component
      data: node,
      // the component to use for rendering. Will get batched with
      // other nodes that use the same component
      component: require.resolve("./src/printer-components/blog-post.js")
    });
    // put a Printer node into the Gatsby system for later processing
    // this is required.
    createNode(printerNode);
    // make the printer node a child of the Mdx node
    // this ensures that if the parent node gets deleted
    // (like if you have drafts that get removed from the node system)
    // this node will get deleted and no processing will happen
    // this is optional
    createParentChildLink({ parent: node, child: printerNode });
  }
};
```

## Manual control

You can also import and use `runScreenshots` but note that you will have to control batching, etc yourself.

```js
import {graphql} from 'gatsby';

exports.onPostBuild = () => {

  const data = await graphql(`
    {
      allBlogPost {
        nodes {
          title
        }
      }
    }
  `).then(r => {
    if (r.errors) {
      reporter.error(r.errors.join(`, `));
    }
    return r.data;
  });

const titles = data.allBlogPost.nodes.map(({ title }) => ({
	id: slugify(title),
    title,
  }));

  await runScreenshots({
  	data: titles,
  	component: './src/printer-components',
  	outputDir: 'public/rainbow-og-images'
  });
}
```