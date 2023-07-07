# Next Docs

The headless ui library for building documentation websites.

📘 Learn More: [Documentation](https://next-docs-zeta.vercel.app)

Next Docs Provides:

-   Search (Supported: flexsearch)
-   Breadcrumb, Sidebar, TOC Components
-   Additional utilities (getTOC, buildPageTree)

## Getting Started

Next Docs is built for Next.js App Router. Hence, this guide will use App Router.

It's recommended to use Next Docs with [Tailwind CSS](https://tailwindcss.com) + [Radix UI](https://www.radix-ui.com), and [Contentlayer](https://www.contentlayer.dev) (or any CMS).

### ContentLayer

Follow this [guide](https://www.contentlayer.dev/docs/getting-started) to setup ContentLayer.

```
npm install contentlayer next-contentlayer
```

### Tailwind CSS

Follow this [guide](https://tailwindcss.com/docs/guides/nextjs) to setup Tailwind CSS.

```
npm install clsx postcss tailwindcss @tailwindcss/typography
```

I also prefer [Shadcn UI](https://ui.shadcn.com) If you don't want to write any components.

### Choose a Source

Next Docs has native support for ContentLayer, but any kind of formats and sources are allowed.

[Learn More](https://next-docs-zeta.vercel.app/docs/adapters/contentlayer)

### Build Page Tree

A page tree refers to structured data of all pages.

```ts
import type { TreeNode } from "next-docs/server";

const tree: TreeNode[] = [
    {
        type: "folder",
        name: "Components",
        url: "/docs/components",
        index: {
            type: "page",
            name: "Quick Start",
            url: "/docs/components",
        },
    },
    {
        type: "separator",
        name: "Guides",
    },
    {
        type: "page",
        name: "Example",
        url: "/docs/example",
    },
];
```

You can convert the data fetched from any CMS to a page tree and pass into Next Docs's components.

Moreover, It supports Contentlayer natively:

```ts
import { allDocs, allMeta } from "contentlayer/generated";
import { buildPageTree } from "next-docs/contentlayer";

export const tree = buildPageTree(allMeta, allDocs);
```

---

### Create Root Layout

implementation of certain components won't be shown but you will learn it later.

```tsx
import { ReactNode } from "react";
import { SidebarProvider, Sidebar } from "@/components/sidebar";
import clsx from "clsx";
import { tree } from "@/utils/page-tree";

export default function DocsLayout({ children }: { children: ReactNode }) {
    return (
        <SidebarProvider>
            <div className="absolute inset-0 -z-[1] overflow-hidden">
                <div className="absolute top-0 left-0 w-full h-[500px] bg-gradient-to-br from-purple-400/20 to-background to-50%" />
            </div>
            <div
                className={clsx(
                    "grid grid-cols-1 gap-12 w-full container max-w-[1400px]",
                    "lg:grid-cols-[250px_auto] xl:grid-cols-[250px_auto_150px] 2xl:grid-cols-[250px_auto_250px]",
                    "sm:px-14 xl:px-24"
                )}
            >
                <Sidebar items={tree} />
                {children}
            </div>
        </SidebarProvider>
    );
}
```

### Create Page

Create `/app/docs/[[...slug]]/page.tsx`:

```tsx
import { allDocs } from "contentlayer/generated";
import { notFound } from "next/navigation";

import { getTableOfContents } from "next-docs/server";
import { getMDXComponent } from "next-contentlayer/hooks";
import { tree } from "@/utils/page-tree";
import React from "react";
import { Breadcrumb } from "@/components/breadcrumb";
import { SafeLink } from "next-docs/link";
import { TOC } from "@/components/toc";

export default async function Page({
    params,
}: {
    params: { slug?: string[] };
}) {
    const path = (params.slug ?? []).join("/");
    const page = allDocs.find((page) => page.slug === path);

    if (page == null) {
        notFound();
    }

    const toc = await getTableOfContents(page.body.raw);
    const MDX = getMDXComponent(page.body.code);

    return (
        <>
            <article className="flex flex-col gap-6 py-8 overflow-x-hidden lg:py-16">
                <Breadcrumb tree={tree} />
                <h1 className="text-4xl font-bold">{page.title}</h1>
                <div className="prose prose-text prose-pre:grid prose-pre:border-[1px] prose-code:bg-secondary prose-code:p-1 max-w-none">
                    <MDX
                        components={{
                            a: SafeLink, // handles external links correctly
                        }}
                    />
                </div>
            </article>
            <div className="relative flex flex-col gap-3 max-xl:hidden py-16">
                <div className="sticky top-28 flex flex-col gap-3 overflow-auto max-h-[calc(100vh-4rem-3rem)]">
                    {toc.length > 0 && (
                        <h3 className="font-semibold">On this page</h3>
                    )}
                    <TOC items={toc} />
                </div>
            </div>
        </>
    );
}
```

You can generate static params as well:

```ts
export async function generateStaticParams() {
    return allDocs.map((docs) => ({
        slug: docs.slug.split("/"),
    }));
}
```

### Create MDX file

Create `/content/docs/index.mdx`:

```mdx
---
title: Quick Start
description: My cool docs
---

Hello World
```

### Enjoy!

We've just created the prototype of the website but many components are missing.

**Next Docs** offers simple document searching as well as components for building a good docs. You can go to our [website](https://next-docs-zeta.vercel.app/docs) to learn more about this!