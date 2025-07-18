<script lang="ts">
	import { box, mergeProps } from "svelte-toolbelt";
	import type { MenuContentStaticProps } from "../types.js";
	import { MenuContentState } from "../menu.svelte.js";
	import { createId } from "$lib/internal/create-id.js";
	import { noop } from "$lib/internal/noop.js";
	import PopperLayer from "$lib/bits/utilities/popper-layer/popper-layer.svelte";
	import { getFloatingContentCSSVars } from "$lib/internal/floating-svelte/floating-utils.svelte.js";
	import PopperLayerForceMount from "$lib/bits/utilities/popper-layer/popper-layer-force-mount.svelte";

	const uid = $props.id();

	let {
		id = createId(uid),
		child,
		children,
		ref = $bindable(null),
		loop = true,
		onInteractOutside = noop,
		onEscapeKeydown = noop,
		onCloseAutoFocus: onCloseAutoFocusProp = noop,
		forceMount = false,
		...restProps
	}: MenuContentStaticProps = $props();

	const contentState = MenuContentState.create({
		id: box.with(() => id),
		loop: box.with(() => loop),
		ref: box.with(
			() => ref,
			(v) => (ref = v)
		),
		onCloseAutoFocus: box.with(() => onCloseAutoFocusProp),
	});

	const mergedProps = $derived(
		mergeProps(restProps, contentState.props, {
			style: { outline: "none" },
		})
	);

	function handleInteractOutside(e: PointerEvent) {
		onInteractOutside(e);
		if (e.defaultPrevented) return;
		contentState.parentMenu.onClose();
	}

	function handleEscapeKeydown(e: KeyboardEvent) {
		onEscapeKeydown(e);
		if (e.defaultPrevented) return;
		contentState.parentMenu.onClose();
	}
</script>

{#if forceMount}
	<PopperLayerForceMount
		{...mergedProps}
		{...contentState.popperProps}
		ref={contentState.opts.ref}
		enabled={contentState.parentMenu.opts.open.current}
		onInteractOutside={handleInteractOutside}
		onEscapeKeydown={handleEscapeKeydown}
		trapFocus
		{loop}
		forceMount={true}
		isStatic
		{id}
	>
		{#snippet popper({ props })}
			{@const finalProps = mergeProps(props, {
				style: {
					outline: "none",
					...getFloatingContentCSSVars("menu"),
				},
			})}
			{#if child}
				{@render child({ props: finalProps, ...contentState.snippetProps })}
			{:else}
				<div {...finalProps}>
					{@render children?.()}
				</div>
			{/if}
		{/snippet}
	</PopperLayerForceMount>
{:else if !forceMount}
	<PopperLayer
		{...mergedProps}
		{...contentState.popperProps}
		ref={contentState.opts.ref}
		open={contentState.parentMenu.opts.open.current}
		onInteractOutside={handleInteractOutside}
		onEscapeKeydown={handleEscapeKeydown}
		trapFocus
		{loop}
		forceMount={false}
		isStatic
		{id}
	>
		{#snippet popper({ props })}
			{@const finalProps = mergeProps(props, {
				style: {
					outline: "none",
					...getFloatingContentCSSVars("menu"),
				},
			})}
			{#if child}
				{@render child({ props: finalProps, ...contentState.snippetProps })}
			{:else}
				<div {...finalProps}>
					{@render children?.()}
				</div>
			{/if}
		{/snippet}
	</PopperLayer>
{/if}
