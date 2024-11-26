<script lang="ts">
	import { goto } from '$app/navigation';
	import { createElementId, determinFileType, getFilenameFromKey } from '$lib/utils.js';
	import {
		Button,
		Collapse,
		Container,
		Icon,
		Image,
		Nav,
		Navbar,
		NavbarBrand,
		NavbarToggler,
		NavItem,
		NavLink,
		Spinner
	} from '@sveltestrap/sveltestrap';

	import 'vidstack/bundle';

	const { data } = $props();

	let filetype: string | false = $state('');
	let isOpen = $state(false);
	let isImageLoaded = $state(false);

	function handleUpdate(event: CustomEvent<boolean>) {
		isOpen = event.detail;
	}

	interface BreadcrumbData {
		name: string;
		prefix: string;
	}

	function createBreadcrumb(path?: string): BreadcrumbData[] {
		let output = [];
		output.push({
			name: '<root>',
			prefix: ''
		});

		if (!path) {
			return output;
		}

		let parts = path?.split('/');

		for (let i = 0; i < parts?.length; i++) {
			const prefix = parts.slice(0, i + 1).join('/');

			output.push({
				name: parts[i],
				prefix: prefix
			});
		}

		return output;
	}

	let breadcrumbData: BreadcrumbData[] = $state([]);
	let path: string = $state('');

	$effect(() => {
		path = data.current ? data.current : '';
		breadcrumbData = createBreadcrumb(path);
		filetype = determinFileType(data.current);
	});
</script>

<svelte:head>
	<title>Gallery - View: {data.current}</title>
</svelte:head>

{#if filetype == 'image'}
	<Container>
		<div class="position-absolute top-0 start-0 h-100 w-100" style="padding-top:10em;">
			{#if !isImageLoaded}
				<div class="position-absolute top-50 start-50 translate-middle">
					<Spinner type="border"></Spinner> Loading
				</div>
			{/if}

			<div class="position-absolute top-0 start-0 h-100 w-100">
				<Image
					src="/get/image/{path}"
					class="h-100 w-100"
					style="object-fit:contain; padding-top:6em;"
					onload={() => (isImageLoaded = true)}
				></Image>
			</div>

			<Button
				color="link"
				outline
				class="position-absolute top-0 start-0 h-100"
				style="width:20%;"
				onclick={() => {
					if (data.previous != null) goto(`/view/${data.previous}`);
				}}
			>
				<Icon name="chevron-left"></Icon>
			</Button>
			<Button
				color="link"
				outline
				class="position-absolute top-0 end-0 h-100 w-10"
				style="width:20%;"
				onclick={() => {
					if (data.next != null) goto(`/view/${data.next}`);
				}}
			>
				<Icon name="chevron-right"></Icon>
			</Button>
		</div>
	</Container>
{/if}

<Container class="sticky-top text-bg-light">
	<Navbar dark expand="md" container="md">
		<NavbarBrand href="/">Gallery</NavbarBrand>
		<NavbarToggler on:click={() => (isOpen = !isOpen)} />
		<Collapse {isOpen} navbar expand="md" on:update={handleUpdate}>
			<Nav class="ms-auto" navbar>
				<NavItem>
					<NavLink href="/get/file/{path}" target="_blank">
						<Icon name="download"></Icon>&nbsp;Get
					</NavLink>
				</NavItem>
			</Nav>
			<Nav navbar>
				<NavItem>
					<NavLink disabled={data.previous == null} href={`/view/${data.previous}`}>
						<Icon name="chevron-left"></Icon>&nbsp;Previous
					</NavLink>
				</NavItem>
				<NavItem>
					<NavLink disabled={data.next == null} href={`/view/${data.next}`}>
						<div class="d-md-none"><Icon name="chevron-right"></Icon>&nbsp;Next</div>
						<div class="d-none d-md-block">Next&nbsp;<Icon name="chevron-right"></Icon></div>
					</NavLink>
				</NavItem>
			</Nav>
		</Collapse>
	</Navbar>
	<nav aria-label="breadcrumb">
		<ol class="breadcrumb">
			{#each breadcrumbData as b, i}
				{#if i == breadcrumbData.length - 1}
					<li class="breadcrumb-item active" aria-current="page">
						{b.name}
					</li>
				{:else}
					<li class="breadcrumb-item">
						<a href="/browse/{b.prefix}#{createElementId(breadcrumbData[i + 1].name)}"
							>{b.name}
						</a>
					</li>
				{/if}
			{/each}
		</ol>
	</nav>
</Container>

{#if filetype == 'video' || filetype == 'audio'}
	<Container>
		<media-player title={getFilenameFromKey(data.current, 'media')} src="/get/file/{path}">
			<media-provider></media-provider>
			<media-video-layout></media-video-layout>
			<media-audio-layout></media-audio-layout>
		</media-player>
	</Container>
{/if}
