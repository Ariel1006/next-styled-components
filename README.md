# next-styled-components

[![NPM Version](https://img.shields.io/npm/v/next-styled-components)](https://npmjs.com/package/next-styled-components)

**Styled-Components that are compatible with Next.js server components (RSC).**

<img src="logo.png" width="250" />

The goal of this library is that you should be able to use it as the [original library](https://github.com/styled-components/styled-components) and it will work with not only client components but also new server ones (RSC).

This library covers most important parts of the original lib API:

- styled(Component)
- styled.element
- createGlobalStyle
- keyframes
- `<ThemeProvider />`
- `<StyleSheetManager />`

There are still some things that have not yet been ported from the original lib. Also there are some things that work slightly differently or currently are hard to implement because of limitations of RSCs.

## How to install

```bash
npm i next-styled-components
```

## How to use it

The most important part is to surround all your styled components with `<StyleSheetManager />`. This component takes care of gathering all styles on the server and on the client and placing the css styles within HTML tree.

Example [`src/app/layout.tsx`](example-nextjs/src/app/layout.tsx):

```typescript
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import { createGlobalStyle, StyleSheetManager, ThemeProvider } from '@/react-component-lib';

const inter = Inter({ subsets: ['latin'] });

const GlobalStyle = createGlobalStyle`
  html, body {
    color: white;
    background-color: black;
  }

  *, *::before, *::after {
    box-sizing: border-box;
  }
`;

export const metadata: Metadata = {
  title: 'Create Next App',
  description: 'Generated by create next app',
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <StyleSheetManager>
          <ThemeProvider theme={{ primaryColor: 'blue' }}>
            <GlobalStyle />
            {children}
          </ThemeProvider>
        </StyleSheetManager>
      </body>
    </html>
  );
}
```

Example [`src/app/page.tsx`](example-nextjs/src/app/page.tsx):

```typescript
import { css, keyframes, styled } from '../react-component-lib';
import { Client } from './client';

const textCss = css`
  text-transform: capitalize;
  text-decoration: underline;
  font-style: italic;
`;

const fadeInOutAnimation = keyframes`
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
`;

const Secondary = styled.span.attrs<{ $isOn: boolean }>(() => ({ role: 'alert' }))`
  display: block;
  width: 100px;
  height: 100px;
  background-color: ${({ theme }) => theme.primaryColor};
  animation: ${fadeInOutAnimation} 1s infinite alternate;
`;

const Main = styled.div<{ color: string }>`
  width: 150px;
  height: 150px;
  padding: 10px;
  background-color: ${({ color }) => color};
  position: relative;

  ${textCss};

  &:hover {
    background-color: yellow;
  }

  & > ${Secondary} {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
`;

export default function Home() {
  return (
    <>
      <Main color="red">
        Server Component
        <Secondary $isOn />
      </Main>
      <Client />
    </>
  );
}
```

And an example of some client component [`src/app/client.tsx`](example-nextjs/src/app/client.tsx):

```typescript
'use client';

import { styled } from '@/react-component-lib';
import { useState } from 'react';

const StyledDiv = styled.div<{ $isOn: boolean }>`
  width: 150px;
  height: 150px;
  padding: 10px;
  background-color: ${({ $isOn }) => ($isOn ? 'pink' : 'green')};
`;

export const Client = () => {
  const [isOn, setIsOnClick] = useState(false);

  return (
    <StyledDiv $isOn={isOn} onClick={() => setIsOnClick((prev) => !prev)}>
      Client Component
    </StyledDiv>
  );
};
```

And if you want to type your theme create a type file, for example [`src/next-styled-components.d.ts`](example-nextjs/src/next-styled-components.d.ts):

```typescript
import 'next-styled-components';

declare module 'next-styled-components' {
  export interface DefaultTheme {
    primaryColor: string;
  }
}
```

## Final note

This library is still in an early development phase.

If you have some ideas how to improve the library, open an Issue or PR. Also, you can directly write to me:

[https://github.com/michal-wrzosek/](https://github.com/michal-wrzosek/)

michal@wrzosek.pl

---

This library was built using [react-component-lib](https://github.com/michal-wrzosek/react-component-lib)
