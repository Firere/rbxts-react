<p align="center">
  <p align="center">
    <img width="150" height="150" src="https://github.com/littensy/rbxts-react/blob/main/images/logo.png?raw=true" alt="Logo">
  </p>
  <h1 align="center"><b>@firere/react</b></h1>
  <p align="center">
    TypeScript type definitions for React Lua.
    <br />
    <a href="https://npmjs.com/package/@rbxts/react"><strong>npm package →</strong></a>
  </p>
</p>

TypeScript type definitions for [React Lua](https://github.com/jsdotlua/react-lua) and [roblox-ts](https://roblox-ts.com), sourced from the official React types. Currently, only [`@rbxts/react`](https://npmjs.com/package/@rbxts/react) and
[`@rbxts/react-roblox`](https://npmjs.com/package/@rbxts/react-roblox) are available.

> [!IMPORTANT]
> This package uses unreleased roblox-ts features, and requires `roblox-ts@next` to be installed.
> If you're encountering issues with `@firere/react`, please see the [Troubleshooting](#-troubleshooting) section for more information.

## 📦 Setup

### Installation

Get started by adding React and ReactRoblox to your project:

```sh
npm install @rbxts/react @rbxts/react-roblox
yarn add @rbxts/react @rbxts/react-roblox
pnpm add @rbxts/react @rbxts/react-roblox # 🔴 See below
```

`roblox-ts` must also be installed with the `next` tag:

```sh
npm install -D roblox-ts@next
yarn add -D roblox-ts@next
pnpm add -D roblox-ts@next
```

Finally, install `@firere/react-vendor` using an alias:
```sh
npm install @rbxts/react-vendor@npm:@firere/react-vendor
yarn add @rbxts/react-vendor@npm:@firere/react-vendor
pnpm add @rbxts/react-vendor@npm:@firere/react-vendor
```

### Configuration

Set up your `tsconfig.json` to use the React JSX factory.

```json
"compilerOptions": {
  "jsxFactory": "React.createElement",
  "jsxFragmentFactory": "React.Fragment"
}
```

### Usage with PNPM

If you're using PNPM as your package manager, you'll need to create a `.npmrc` file in the root of your project with the following content:

```ini
node-linker=hoisted
```

## 🚀 Examples

### Mounting your app

```tsx
import React, { StrictMode } from "@rbxts/react";
import { createPortal, createRoot } from "@rbxts/react-roblox";

const root = createRoot(new Instance("Folder"));

root.render(<StrictMode>{createPortal(<App />, playerGui)}</StrictMode>);
```

### Function Component

```tsx
import React, { useState } from "@rbxts/react";

interface CounterProps {
	initialCount: number;
}

export function Counter({ initialCount }: CounterProps) {
	const [count, setCount] = useState(initialCount);

	return (
		<textbutton
			Text={`Count: ${count}`}
			AnchorPoint={new Vector2(0.5, 0.5)}
			Size={new UDim2(0, 100, 0, 50)}
			Position={new UDim2(0.5, 0, 0.5, 0)}
			Event={{
				Activated: () => setCount(count + 1),
			}}
		/>
	);
}
```

### Class Component

```tsx
import React, { Component, ReactComponent } from "@rbxts/react";

interface CounterProps {
	initialCount: number;
}

interface CounterState {
	count: number;
}

@ReactComponent
export class Counter extends Component<CounterProps, CounterState> {
	state: CounterState = {
		count: this.props.initialCount,
	};

	render() {
		return (
			<textbutton
				Text={`Count: ${this.state.count}`}
				AnchorPoint={new Vector2(0.5, 0.5)}
				Size={new UDim2(0, 100, 0, 50)}
				Position={new UDim2(0.5, 0, 0.5, 0)}
				Event={{
					Activated: () => this.setState({ count: this.state.count + 1 }),
				}}
			/>
		);
	}
}
```

### Error Boundary

```tsx
import React, { Component, ErrorInfo, ReactComponent } from "@rbxts/react";

interface ErrorBoundaryProps {
	fallback: (error: unknown) => React.Element;
}

interface ErrorBoundaryState {
	hasError: boolean;
	message?: unknown;
}

@ReactComponent
export class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
	state: ErrorBoundaryState = {
		hasError: false,
	};

	componentDidCatch(message: unknown, info: ErrorInfo) {
		warn(message, info.componentStack);

		this.setState({
			hasError: true,
			message: `${message} ${info.componentStack}`,
		});
	}

	render() {
		if (this.state.hasError) {
			return this.props.fallback(this.state.message);
		} else {
			return this.props.children;
		}
	}
}
```

## 🔎 Troubleshooting

### compilerOptions.jsxFactory must be `Roact.createElement`!

If you encounter this error during compilation, it means that you're using an older version of `roblox-ts`. Make sure to install `roblox-ts@next` and uninstall any global installations of `roblox-ts`:

```sh
npm uninstall -g roblox-ts
npm install -D roblox-ts@next
```

### `(X)` cannot be used as a JSX component. Its return type `Element` is not a valid JSX element.

This error occurs when a conflicting installation of react-ts is present in your project. This can be for one of two reasons:

1. You have react-ts installed in your dependencies. (`npm uninstall @rbxts/roact`)
2. You have an outdated package installed that depends on react-ts.

The most common cause is an outdated package. To view the packages that depend on `@rbxts/react-ts` (which will be under the alias `@rbxts/roact`), run the following command:

```sh
npm ls @rbxts/roact
```

If you find any packages that depend on `@rbxts/react-ts`, you should update them to the latest version, or open an issue on their repository to request an update.

### Attempt to index nil with `useMemo` (or other hooks)

This error happens when React can't figure out how to retrieve the current instance of the component. This can be for a number of reasons:

1. You're using a hook outside of a component, or using function components incorrectly.

Hooks must be used inside the body of a function component. A common mistake is to call hooks conditionally, or inside a callback function. Make sure you're calling hooks at the top level of your function component.

**Do not call a function component directly.** To render a function component, wrap it in a JSX tag:

```tsx
<App />; // 🟢 Good
App(); // 🔴 Bad
```

2. There's more than one version of React in your project.

When multiple versions of React are present in your node_modules, any packages that depend on React might try to use the wrong one.

Make sure your `rbxts_include.node_module.@rbxts` folder in Roblox Studio doesn't contain a module named "React" (capital R). This module is a sign Rojo has not fully removed react-ts. If so, you should delete your `node_modules` folder and restart Rojo.

If a fresh install doesn't fix the issue, you might have a package installed that depends on react-ts. See [the previous section](#x-cannot-be-used-as-a-jsx-component-its-return-type-element-is-not-a-valid-jsx-element) for more information.

### My issue isn't listed here!

If you're encountering an issue that isn't listed here, please [post your issue](https://discord.com/channels/476080952636997633/1006309509876162570) in the [roblox-ts Discord server](https://discord.roblox-ts.com/).

## 📚 Resources

- [React Documentation](https://react.dev) - Learn about React's core concepts and API
- [React Lua Documentation](https://jsdotlua.github.io/react-lua/) - A comprehensive guide for the differences between Roact and React
- [JS.Lua Repository](https://github.com/jsdotlua/react-lua) - The source code for React Lua

## 📝 License

This project is licensed under the [MIT license](LICENSE).

## Why does this exist?
Back in the days of Legacy Roact, I had [my own version](https://github.com/Firere/roact) which automatically [skipped over applying invalid props](https://firere.github.io/roact/guide/invalid-prop-discarding) and which merged children for you, as [Roact itself suggested](https://firere.github.io/roact/guide/child-merging/#:~:text=The%20prop,fullProps).

This was so I could write my components like:
```tsx
interface Props extends React.InstanceProps<TextButton> {
	someProp: boolean;
}

function SomeComponent(props: Props) {
	// pretend there's some logic involving someProp here

	return (
		<textbutton BackgroundColor3={new Color3()} Text="Button" {...props}>
			<textlabel Text="Ooh, look, a child!" />
		</textbutton>
	);
}

export default SomeComponent;
```
Sweet! Now in `SomeComponent`, I can easily overwrite its default props, and give it more children when using it if need be without the hassle of needing to create a utility function.
```tsx
<SomeComponent BackgroundColor3={new Color3(1, 1, 1)} someProp={false} Text="Special button">
	<textlabel Text="Yes! The more children, the merrier! I love children!" />
</SomeComponent>
```

Anyway, why modify Roact and not just do [what everyone else is doing](https://blog.boyned.com/articles/things-i-learned-using-react/#:~:text=Traditionally)? Simple! I didn't have any idea what everyone else was doing. None of my friends are Roblox devs. I didn't even use `local e = Roact.createElement` before I switched to TypeScript.

All this particular version is is me carrying some of the code from my old Roact modification. Emphasis on *some*: it no longer supports child merging (as it was working really strangely when I did carry that over - I had to wrap certain children of components in fragments for them to render, for some reason) or configuration (as I don't intend nor expect anyone else to use this, along with the fact the only thing this now does is skip applying invalid props).

The changes (by which I mean 2 additional lines of code) can be found in `react-roblox > client > roblox > RobloxComponentProps`. Use it if you want. I'll probably only bother updating this to any latest version if upstream ever gets an update of around the calibre of v19. (God, I'd really love Roact v19. I know there isn't any way for Roact to do it since it's in Luau and there needs to be a compiler, but maybe roblox-ts can cook something up?)
