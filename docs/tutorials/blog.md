---
title: Developer Blog
order: 1
---

# Quickstart

We're going to be short on words and quick on code in this quickstart. If you're looking to see what Remix is all about in 15 minutes, this is it.

<docs-info>💿 Hey I'm Derrick the Remix Compact Disc 👋 Whenever you're supposed to _do_ something you'll see me</docs-info>

This tutorial uses TypeScript. Remix can definitely be used without TypeScript. We feel most productive when writing TypeScript, but if you'd prefer to skip the TypeScript syntax, feel free to write your code in JavaScript.

## Prerequisites

If you want to follow this tutorial locally on your own computer, it is important for you to have these things installed:

- [Node.js](https://nodejs.org) 14 or greater
- [npm](https://www.npmjs.com) 7 or greater
- A code editor

## Creating the project

<docs-warning>Make sure you are running at least Node v14 or greater</docs-warning>

💿 Initialize a new Remix project. We'll call ours "blog-tutorial" but you can call it something else if you'd like.

```sh
npx create-remix --template remix-run/indie-stack blog-tutorial
```

```
? Do you want me to run `npm install`? Yes
...
? Do you want to run the build/tests/etc to verify things are setup properly? Yes
```

<docs-info>Running the verify script is optional, but handy.</docs-info>

You can read more about the stacks available in [the stacks docs](/pages/stacks).

We're using [the Indie stack](https://github.com/remix-run/indie-stack), which is a full application ready to deploy to [fly.io](https://fly.io). This includes development tools as well as production-ready authentication and persistence. Don't worry if you're unfamiliar with the tools used, we'll walk you through things as we go.

💿 Now, open the project that was generated in your preferred editor and check the instructions in the `README.md` file. Feel free to read over this. We'll get to the deployment bit later in the tutorial.

💿 Let's start the dev server:

```sh
npm run dev
```

💿 Open up [http://localhost:3000](http://localhost:3000), the app should be running.

If you want, take a minute and poke around the UI a bit. Feel free to create an account and create/delete some notes to get an idea of what's available in the UI out of the box.

## Your First Route

We're going to make a new route to render at the "/posts" URL. Before we do that, let's link to it.

💿 Add a link to posts in `app/routes/index.tsx`

Go ahead and copy/paste this:

```tsx
<div className="mx-auto mt-16 max-w-7xl text-center">
  <Link
    to="/posts"
    className="text-xl text-blue-600 underline"
  >
    Blog Posts
  </Link>
</div>
```

You can put it anywhere you like. I stuck it right above the icons of all the technologies used in the stack:

![Screenshot of the app showing the blog post link](/blog-tutorial/blog-post-link.png)

<docs-info>You may have noticed we're using <a href="https://tailwindcss.com">tailwind</a> classes.</docs-info>

The Remix Indie stack has [tailwind](https://tailwindcss.com) support pre-configured. If you'd prefer to not use tailwind, you're welcome to remove it and use something else. Learn more about your styling options with Remix in [the styling guide](/guides/styling).

Back in the browser go ahead and click the link. You should see a 404 page since we've not created this route yet. Let's create the route now:

💿 Create a new file in `app/routes/posts/index.tsx`

```sh
mkdir app/routes/posts
touch app/routes/posts/index.tsx
```

<docs-info>Any time you see terminal commands to create files or folders, you can of course do that however you'd like, but using `mkdir` and `touch` is just a way for us to make it clear which files you should be creating.</docs-info>

We could have named it just `posts.tsx` but we'll have another route soon and it'll be nice to put them by each other. An index route will render at the folder's path (just like index.html on a web server).

Now if you navigate to the `/posts` route, you'll get an error indicating there's no way to handle the request. That's because we haven't done anything in that route yet! Let's add a component and export it as the default:

💿 Make the posts component

```tsx filename=app/routes/posts/index.tsx
export default function Posts() {
  return (
    <main>
      <h1>Posts</h1>
    </main>
  );
}
```

You might need to refresh the browser to see our new, bare-bones posts route.

## Loading Data

Data loading is built into Remix.

If your web dev background is primarily in the last few years, you're probably used to creating two things here: an API route to provide data and a frontend component that consumes it. In Remix your frontend component is also its own API route and it already knows how to talk to itself on the server from the browser. That is, you don't have to fetch it.

If your background is a bit farther back than that with MVC web frameworks like Rails, then you can think of your Remix routes as backend views using React for templating, but then they know how to seamlessly hydrate in the browser to add some flair instead of writing detached jQuery code to dress up the user interactions. It's progressive enhancement realized in its fullest. Additionally, your routes are their own controller.

So let's get to it and provide some data to our component.

💿 Make the posts route "loader"

```tsx filename=app/routes/posts/index.tsx lines=[1,3-16,19-20]
import { json, useLoaderData } from "remix";

export const loader = async () => {
  return json({
    posts: [
      {
        slug: "my-first-post",
        title: "My First Post",
      },
      {
        slug: "90s-mixtape",
        title: "A Mixtape I Made Just For You",
      },
    ],
  });
};

export default function Posts() {
  const { posts } = useLoaderData();
  console.log(posts);
  return (
    <main>
      <h1>Posts</h1>
    </main>
  );
}
```

Loaders are the backend "API" for their component and it's already wired up for you through `useLoaderData`. It's a little wild how blurry the line is between the client and the server in a Remix route. If you have your server and browser consoles both open, you'll note that they both logged our post data. That's because Remix rendered on the server to send a full HTML document like a traditional web framework, but it also hydrated in the client and logged there too.

💿 Render links to our posts

```tsx filename=app/routes/posts/index.tsx lines=[1,9-20] nocopy
import { Link, json, useLoaderData } from "remix";

// ...
export default function Posts() {
  const { posts } = useLoaderData();
  return (
    <main>
      <h1>Posts</h1>
      <ul>
        {posts.map((post: any) => (
          <li key={post.slug}>
            <Link
              to={post.slug}
              className="text-blue-600 underline"
            >
              {post.title}
            </Link>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

TypeScript is mad, so let's help it out:

💿 Add the Post type and generic for `useLoaderData`

```tsx filename=app/routes/posts/index.tsx lines=[3-6,8-10,13,28]
import { Link, json, useLoaderData } from "remix";

type Post = {
  slug: string;
  title: string;
};

type LoaderData = {
  posts: Array<Post>;
};

export const loader = async () => {
  return json<LoaderData>({
    posts: [
      {
        slug: "my-first-post",
        title: "My First Post",
      },
      {
        slug: "90s-mixtape",
        title: "A Mixtape I Made Just For You",
      },
    ],
  });
};

export default function Posts() {
  const { posts } = useLoaderData() as LoaderData;
  return (
    <main>
      <h1>Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.slug}>
            <Link
              to={post.slug}
              className="text-blue-600 underline"
            >
              {post.title}
            </Link>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

Hey, that's pretty cool. We get a pretty solid degree of type safety even over a network request because it's all defined in the same file. Unless the network blows up while Remix fetches the data, you've got type safety in this component and its API (remember, the component is already its own API route).

## A little refactoring

A solid practice is to create a module that deals with a particular concern. In our case it's going to be reading and writing posts. Let's set that up now and add a `getPosts` export to our module.

💿 Create `app/models/post.server.ts`

```sh
touch app/models/post.server.ts
```

We're mostly gonna copy/paste stuff from our route:

```tsx filename=app/models/post.server.ts
type Post = {
  slug: string;
  title: string;
};

export async function getPosts(): Promise<Array<Post>> {
  return [
    {
      slug: "my-first-post",
      title: "My First Post",
    },
    {
      slug: "90s-mixtape",
      title: "A Mixtape I Made Just For You",
    },
  ];
}
```

Note that we're making the `getPosts` function `async` because even though it's not currently doing anything async it will soon!

💿 Update the posts route to use our new posts module:

```tsx filename=app/routes/posts/index.tsx nocopy
import { Link, json, useLoaderData } from "remix";
import { getPosts } from "~/models/post.server";

type LoaderData = {
  // this is a handy way to say: "posts is whatever type getPosts resolves to"
  posts: Awaited<ReturnType<typeof getPosts>>;
};

export const loader = async () => {
  return json<LoaderData>({
    posts: await getPosts(),
  });
};

// ...
```

## Pulling from a data source

With the Indie Stack, we've got a SQLite database already set up and configured for us, so let's update our Database Schema to handle SQLite. We're using [Prisma](https://prisma.io) to interact with the database, so we'll update that schema and Prisma will take care of updating our database to match the schema for us (as well as generating and running the necessary SQL commands for the migration).

<docs-info>You do not have to use Prisma when using Remix. Remix works great with whatever existing database or data persistence services you're currently using.</docs-info>

If you've never used Prisma before, don't worry, we'll walk you through it.

💿 First, we need to update our Prisma schema:

```prisma filename=prisma/schema.prisma nocopy
// Stick this at the bottom of that file:

model Post {
  slug     String @id
  title    String
  markdown String

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

💿 Now let's tell Prisma to update our local database and TypeScript definitions to match this schema change:

```sh
npx prisma db push
```

💿 Let's seed our database with a couple posts. Open `prisma/seed.ts` and add this to the end of the seed functionality (right before the `console.log`):

```ts filename=prisma/seed.ts
const posts = [
  {
    slug: "my-first-post",
    title: "My First Post",
    markdown: `
# This is my first post

Isn't it great?
    `.trim(),
  },
  {
    slug: "90s-mixtape",
    title: "A Mixtape I Made Just For You",
    markdown: `
# 90s Mixtape

- I wish (Skee-Lo)
- This Is How We Do It (Montell Jordan)
- Everlong (Foo Fighters)
- Ms. Jackson (Outkast)
- Interstate Love Song (Stone Temple Pilots)
- Killing Me Softly With His Song (Fugees, Ms. Lauryn Hill)
- Just a Friend (Biz Markie)
- The Man Who Sold The World (Nirvana)
- Semi-Charmed Life (Third Eye Blind)
- ...Baby One More Time (Britney Spears)
- Better Man (Pearl Jam)
- It's All Coming Back to Me Now (Céline Dion)
- This Kiss (Faith Hill)
- Fly Away (Lenny Kravits)
- Scar Tissue (Red Hot Chili Peppers)
- Santa Monica (Everclear)
- C'mon N' Ride it (Quad City DJ's)
    `.trim(),
  },
];

for (const post of posts) {
  await prisma.post.upsert({
    where: { slug: post.slug },
    update: post,
    create: post,
  });
}
```

<docs-info>Note that we're using `upsert` so you can run the seed script over and over without adding multiple versions of the same post every time.</docs-info>

Great, let's get those posts into the database with the seed script:

```
npx prisma db seed
```

💿 Now update the `app/models/post.server.ts` file to read from the SQLite database:

```ts filename=app/models/post.server.ts
import { prisma } from "~/db.server";

export async function getPosts() {
  return prisma.post.findMany();
}
```

<docs-success>Notice we're able to remove the return type, but everything is still fully typed. The TypeScript feature of Prisma is one of its greatest strengths. Less manual typing, but still type safe!</docs-success>

<docs-info>The `~/db.server` import is importing the file at `app/db.server.ts`. The `~` is a fancy alias to the `app` directory so you don't have to worry about how many `../../`s to include in your import as you move files around.</docs-info>

💿 Now that the Prisma client has been updated, we will need to restart our server. So stop the dev server and start it back up again with `npm run dev`.

<docs-warning>You only need to ever do this when you change the Prisma schema and update the Prisma client. Normally you don't need to restart the dev server during development. Nice that it's so fast though right?</docs-warning>

With the server up and running again, you should be able to go to `http://localhost:3000/post` and the posts should still be there, but now they're coming from SQLite!

## Dynamic Route Params

Now let's make a route to actually view the post. We want these URLs to work:

```
/posts/my-first-post
/posts/90s-mixtape
```

Instead of creating a route for every single one of our posts, we can use a "dynamic segment" in the url. Remix will parse and pass to us so we can look up the post dynamically.

💿 Create a dynamic route at "app/routes/posts/$slug.tsx"

```sh
touch app/routes/posts/\$slug.tsx
```

```tsx filename=app/routes/posts/$slug.tsx
export default function PostSlug() {
  return (
    <main className="mx-auto max-w-4xl">
      <h1 className="my-6 border-b-2 text-center text-3xl">
        Some Post
      </h1>
    </main>
  );
}
```

You can click one of your posts and should see the new page.

💿 Add a loader to access the params

```tsx filename=app/routes/posts/$slug.tsx lines=[1,3-5,8,12]
import { useLoaderData, json } from "remix";

export const loader = async ({ params }) => {
  return json({ slug: params.slug });
};

export default function PostSlug() {
  const { slug } = useLoaderData();
  return (
    <main className="mx-auto max-w-4xl">
      <h1 className="my-6 border-b-2 text-center text-3xl">
        Some Post: {slug}
      </h1>
    </main>
  );
}
```

The part of the filename attached to the `$` becomes a named key on the `params` object that comes into your loader. This is how we'll look up our blog post.

💿 Let's get some help from TypeScript for the loader function signature.

```tsx filename=app/routes/posts/$slug.tsx lines=[1,4]
import type { LoaderFunction } from "remix";
import { json, useLoaderData } from "remix";

export const loader: LoaderFunction = async ({
  params,
}) => {
  return json({ slug: params.slug });
};
```

Now, let's actually get the post contents from the database by it's slug.

💿 Add a `getPost` function to our post module

Update the `app/models/post.server.ts` file:

```tsx filename=app/models/post.server.ts lines=[3,9-11]
import { prisma } from "~/db.server";

export type { Post } from "@prisma/client";

export async function getPosts() {
  return prisma.post.findMany();
}

export async function getPost(slug: string) {
  return prisma.post.findUnique({ where: { slug } });
}
```

💿 Use the new `getPost` function in the route

```tsx filename=app/routes/posts/$slug.tsx lines=[3,8-9,13,17]
import type { LoaderFunction } from "remix";
import { useLoaderData, json } from "remix";
import { getPost } from "~/models/post.server";

export const loader: LoaderFunction = async ({
  params,
}) => {
  const post = await getPost(params.slug);
  return json({ post });
};

export default function PostSlug() {
  const { post } = useLoaderData();
  return (
    <main className="mx-auto max-w-4xl">
      <h1 className="my-6 border-b-2 text-center text-3xl">
        {post.title}
      </h1>
    </main>
  );
}
```

Check that out! We're now pulling our posts from a data source instead of including it all in the browser as JavaScript.

Let's make TypeScript happy with our code:

```tsx filename=app/routes/posts/$slug.tsx lines=[4-5,7,12,15,17,21]
import type { LoaderFunction } from "remix";
import { useLoaderData, json } from "remix";
import { getPost } from "~/models/post.server";
import type { Post } from "~/models/post.server";
import invariant from "tiny-invariant";

type LoaderData = { post: Post };

export const loader: LoaderFunction = async ({
  params,
}) => {
  invariant(params.slug, `params.slug is required`);

  const post = await getPost(params.slug);
  invariant(post, `Post not found: ${params.slug}`);

  return json<LoaderData>({ post });
};

export default function PostSlug() {
  const { post } = useLoaderData() as LoaderData;
  return (
    <main className="mx-auto max-w-4xl">
      <h1 className="my-6 border-b-2 text-center text-3xl">
        {post.title}
      </h1>
    </main>
  );
}
```

Quick note on that `invariant` for the params. Because `params` comes from the URL, we can't be totally sure that `params.slug` will be defined--maybe you change the name of the file to `$postId.ts`! It's good practice to validate that stuff with `invariant`, and it makes TypeScript happy too.

We also have an invariant for the post. We'll handle the `404` case better later. Keep going!

Now let's get that markdown parsed and rendered to HTML to the page. There are a lot of markdown parsers, we'll use "marked" for this tutorial because it's really easy to get working.

💿 Parse the markdown into HTML

```sh
npm add marked
# if using typescript
npm add @types/marked -D
```

```tsx filename=app/routes/post/$slug.ts lines=[6,8,18-19,23,29]
import type { LoaderFunction } from "remix";
import { useLoaderData, json } from "remix";
import { getPost } from "~/models/post.server";
import type { Post } from "~/models/post.server";
import invariant from "tiny-invariant";
import { marked } from "marked";

type LoaderData = { post: Post; html: string };

export const loader: LoaderFunction = async ({
  params,
}) => {
  invariant(params.slug, `params.slug is required`);

  const post = await getPost(params.slug);
  invariant(post, `Post not found: ${params.slug}`);

  const html = marked(post.markdown);
  return json<LoaderData>({ post, html });
};

export default function PostSlug() {
  const { post, html } = useLoaderData() as LoaderData;
  return (
    <main className="mx-auto max-w-4xl">
      <h1 className="my-6 border-b-2 text-center text-3xl">
        {post.title}
      </h1>
      <div dangerouslySetInnerHTML={{ __html: html }} />
    </main>
  );
}
```

Holy smokes, you did it. You have a blog. Check it out! Next, we're gonna make it easier to create new blog posts 📝

## Creating Blog Posts

Right now, our blog posts just come from seeding the database. Not a real solution, so we need a way to create a new blog post in the database. We're going to be using actions for that.

Let's make a new "admin" section of the app.

💿 Create an admin route within the `posts` directory:

```sh
touch app/routes/posts/admin.tsx
```

```tsx filename=app/routes/posts/admin.tsx
import { Link, useLoaderData } from "remix";

import { getPosts } from "~/models/post.server";
import type { Post } from "~/models/post.server";

export const loader = async () => {
  return getPosts();
};

export default function Admin() {
  const posts = useLoaderData<Post[]>();
  return (
    <div className="admin">
      <nav>
        <h1>Admin</h1>
        <ul>
          {posts.map((post) => (
            <li key={post.slug}>
              <Link to={post.slug}>{post.title}</Link>
            </li>
          ))}
        </ul>
      </nav>
      <main>...</main>
    </div>
  );
}
```

You should recognize a lot of that code from the posts route. We set up some extra HTML structure because we're going to style this real quick.

💿 Create an admin stylesheet

```sh
mkdir app/styles
touch app/styles/admin.css
```

```css filename=app/styles/admin.css
.admin {
  display: flex;
}

.admin > nav {
  padding-right: 2rem;
}

.admin > main {
  flex: 1;
  border-left: solid 1px #ccc;
  padding-left: 2rem;
}

em {
  color: red;
}
```

💿 Link to the stylesheet in the admin route

```tsx filename=app/routes/posts/admin.tsx lines=[5,7-9]
import { Link, useLoaderData } from "remix";

import { getPosts } from "~/models/post.server";
import type { Post } from "~/models/post.server";
import adminStyles from "~/styles/admin.css";

export const links = () => {
  return [{ rel: "stylesheet", href: adminStyles }];
};

// ...
```

Each route can export a `links` function that returns array of `<link>` tags, except in object form instead of HTML. So we use `{ rel: "stylesheet", href: adminStyles}` instead of `<link rel="stylesheet" href="..." />`. This allows Remix to merge all of your rendered routes links together and render them in the `<Links/>` element at the top of your document. You can see another example of this in `root.tsx` if you're curious.

Alright, you should have a decent looking page with the posts on the left and a placeholder on the right.
For now, you need to navigate to [http://localhost:3000/admin](http://localhost:3000/admin) manually as we haven't set up any navigational links yet.

## Index Routes

Let's fill in that placeholder with an index route for admin. Hang with us, we're introducing "nested routes" here where your route file nesting becomes UI component nesting.

💿 Create a folder for `admin.tsx`'s child routes, with an index inside

```sh
mkdir app/routes/admin
touch app/routes/admin/index.tsx
```

```tsx filename=app/routes/admin/index.tsx
import { Link } from "remix";

export default function AdminIndex() {
  return (
    <p>
      <Link to="new">Create a New Post</Link>
    </p>
  );
}
```

If you refresh you're not going to see it yet. Every route inside of `app/routes/admin/` can now render _inside_ of `app/routes/posts/admin.tsx` when their URL matches. You get to control which part of the `admin.tsx` layout the child routes render.

💿 Add an outlet to the admin page

```tsx filename=app/routes/posts/admin.tsx lines=[1,21]
import { Outlet, Link, useLoaderData } from "remix";

//...
export default function Admin() {
  const posts = useLoaderData<Post[]>();
  return (
    <div className="admin">
      <nav>
        <h1>Admin</h1>
        <ul>
          {posts.map((post) => (
            <li key={post.slug}>
              <Link to={`/posts/${post.slug}`}>
                {post.title}
              </Link>
            </li>
          ))}
        </ul>
      </nav>
      <main>
        <Outlet />
      </main>
    </div>
  );
}
```

Hang with us for a minute, index routes can be confusing at first. Just know that when the URL matches the parent route's path, the index will render inside the outlet.

Maybe this will help, let's add the "/admin/new" route and see what happens when we click the link.

💿 Create the `app/routes/admin/new.tsx` route

```sh
touch app/routes/admin/new.tsx
```

```tsx filename=app/routes/admin/new.tsx
export default function NewPost() {
  return <h2>New Post</h2>;
}
```

Now click the link from the index route and watch the `<Outlet/>` automatically swap out the index route for the "new" route!

## Actions

We're gonna get serious now. Let's build a form to create a new post in our new "new" route.

💿 Add a form to the new route

```tsx filename=app/routes/admin/new.tsx lines=[1,4-25]
import { Form } from "remix";

export default function NewPost() {
  return (
    <Form method="post">
      <p>
        <label>
          Post Title: <input type="text" name="title" />
        </label>
      </p>
      <p>
        <label>
          Post Slug: <input type="text" name="slug" />
        </label>
      </p>
      <p>
        <label htmlFor="markdown">Markdown:</label>
        <br />
        <textarea id="markdown" rows={20} name="markdown" />
      </p>
      <p>
        <button type="submit">Create Post</button>
      </p>
    </Form>
  );
}
```

If you love HTML like us, you should be getting pretty excited. If you've been doing a lot of `<form onSubmit>` and `<button onClick>` you're about to have your mind blown by HTML.

All you really need for a feature like this is a form to get data from the user and a backend action to handle it. And in Remix, that's all you have to do, too.

Let's create the essential code that knows how to save a post first in our `post.ts` module.

💿 Add `createPost` anywhere inside of `app/post.ts`

```tsx filename=app/post.ts
// ...
export async function createPost(post) {
  const md = `---\ntitle: ${post.title}\n---\n\n${post.markdown}`;
  await fs.writeFile(
    path.join(postsPath, post.slug + ".md"),
    md
  );
  return getPost(post.slug);
}
```

💿 Call `createPost` from the new post route's action

```tsx filename=app/routes/admin/new.tsx lines=[1,3,5-15]
import { redirect, Form } from "remix";

import { createPost } from "~/models/post.server";

export const action = async ({ request }) => {
  const formData = await request.formData();

  const title = formData.get("title");
  const slug = formData.get("slug");
  const markdown = formData.get("markdown");

  await createPost({ title, slug, markdown });

  return redirect("/admin");
};

export default function NewPost() {
  // ...
}
```

That's it. Remix (and the browser) will take care of the rest. Click the submit button and watch the sidebar that lists our posts update automatically.

In HTML an input's `name` attribute is sent over the network and available by the same name on the request's `formData`.

TypeScript is mad again, let's add some types.

💿 Add the types to both files we changed

```tsx filename=app/post.ts lines=[2-6,8]
// ...
type NewPost = {
  title: string;
  slug: string;
  markdown: string;
};

export async function createPost(post: NewPost) {
  const md = `---\ntitle: ${post.title}\n---\n\n${post.markdown}`;
  await fs.writeFile(
    path.join(postsPath, post.slug + ".md"),
    md
  );
  return getPost(post.slug);
}

//...
```

```tsx filename=app/routes/admin/new.tsx lines=[2,6]
import { Form, redirect } from "remix";
import type { ActionFunction } from "remix";

import { createPost } from "~/models/post.server";

export const action: ActionFunction = async ({
  request,
}) => {
  const formData = await request.formData();

  const title = formData.get("title");
  const slug = formData.get("slug");
  const markdown = formData.get("markdown");

  await createPost({ title, slug, markdown });

  return redirect("/admin");
};
```

Whether you're using TypeScript or not, we've got a problem when the user doesn't provide values on some of these fields (and TS is still mad about that call to `createPost`).

Let's add some validation before we create the post.

💿 Validate if the form data contains what we need, and return the errors if not

```tsx filename=app/routes/admin/new.tsx lines=[11-14,16-18]
//...
export const action: ActionFunction = async ({
  request,
}) => {
  const formData = await request.formData();

  const title = formData.get("title");
  const slug = formData.get("slug");
  const markdown = formData.get("markdown");

  const errors = {};
  if (!title) errors.title = true;
  if (!slug) errors.slug = true;
  if (!markdown) errors.markdown = true;

  if (Object.keys(errors).length) {
    return errors;
  }

  await createPost({ title, slug, markdown });

  return redirect("/admin");
};
```

Notice we don't return a redirect this time, we actually return the errors. These errors are available to the component via `useActionData`. It's just like `useLoaderData` but the data comes from the action after a form POST.

💿 Add validation messages to the UI

```tsx filename=app/routes/admin/new.tsx lines=[1,7,13-16,22-23,28-31]
import { useActionData, Form, redirect } from "remix";
import type { ActionFunction } from "remix";

// ...

export default function NewPost() {
  const errors = useActionData();

  return (
    <Form method="post">
      <p>
        <label>
          Post Title:{" "}
          {errors?.title ? (
            <em>Title is required</em>
          ) : null}
          <input type="text" name="title" />
        </label>
      </p>
      <p>
        <label>
          Post Slug:{" "}
          {errors?.slug ? <em>Slug is required</em> : null}
          <input type="text" name="slug" />
        </label>
      </p>
      <p>
        <label htmlFor="markdown">Markdown:</label>{" "}
        {errors?.markdown ? (
          <em>Markdown is required</em>
        ) : null}
        <br />
        <textarea id="markdown" rows={20} name="markdown" />
      </p>
      <p>
        <button type="submit">Create Post</button>
      </p>
    </Form>
  );
}
```

TypeScript is still mad, so let's add some invariants and a new type for the error object to make it happy.

```tsx filename=app/routes/admin/new.tsx lines=[2,4-8,15,24-26]
//...
import invariant from "tiny-invariant";

type PostError = {
  title?: boolean;
  slug?: boolean;
  markdown?: boolean;
};

export const action: ActionFunction = async ({
  request,
}) => {
  // ...

  const errors: PostError = {};
  if (!title) errors.title = true;
  if (!slug) errors.slug = true;
  if (!markdown) errors.markdown = true;

  if (Object.keys(errors).length) {
    return errors;
  }

  invariant(typeof title === "string");
  invariant(typeof slug === "string");
  invariant(typeof markdown === "string");
  await createPost({ title, slug, markdown });

  return redirect("/admin");
};
```

For some real fun, disable JavaScript in your dev tools and try it out. Because Remix is built on the fundamentals of HTTP and HTML, this whole thing works without JavaScript in the browser. But that's not the point. Let's slow this down and add some "pending UI" to our form.

💿 Slow down our action with a fake delay

```tsx filename=app/routes/admin/new.tsx lines=[5-6]
// ...
export const action: ActionFunction = async ({
  request,
}) => {
  await new Promise((res) => setTimeout(res, 1000));

  const formData = await request.formData();

  const title = formData.get("title");
  const slug = formData.get("slug");
  const markdown = formData.get("markdown");
  // ...
};
//...
```

💿 Add some pending UI with `useTransition`

```tsx filename=app/routes/admin/new.tsx lines=[2,12,20-22]
import {
  useTransition,
  useActionData,
  Form,
  redirect,
} from "remix";

// ...

export default function NewPost() {
  const errors = useActionData();
  const transition = useTransition();

  return (
    <Form method="post">
      {/* ... */}

      <p>
        <button type="submit">
          {transition.submission
            ? "Creating..."
            : "Create Post"}
        </button>
      </p>
    </Form>
  );
}
```

Now the user gets an enhanced experience than if we had just done this without JavaScript in the browser at all. Some other things that you could do to make it better is automatically slugify the title into the slug field or let the user override it (maybe we'll add that later).

That's it for today! Your homework is to make an `/admin/edit` page for your posts. The links are already there in the sidebar but they return 404! Create a new route that reads the posts, and puts them into the fields. All the code you need is already in `app/routes/posts/$slug.tsx` and `app/routes/admin/new.tsx`. You just gotta put it together.

We hope you love Remix!
