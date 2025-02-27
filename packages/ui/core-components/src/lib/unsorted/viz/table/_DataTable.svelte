<script>
	import { writable } from 'svelte/store';
	import { setContext } from 'svelte';
	import { slide } from 'svelte/transition';
	import { propKey, strictBuild } from '@evidence-dev/component-utilities/chartContext';
	import getColumnSummary from '@evidence-dev/component-utilities/getColumnSummary';
	import { convertColumnToDate } from '@evidence-dev/component-utilities/dateParsing';
	import ErrorChart from '../core/ErrorChart.svelte';
	import SearchBar from '../core/SearchBar.svelte';
	import checkInputs from '@evidence-dev/component-utilities/checkInputs';
	import DownloadData from '../../ui/DownloadData.svelte';
	import InvisibleLinks from '../../../atoms/InvisibleLinks.svelte';

	import { Icon } from '@steeze-ui/svelte-icon';
	import CodeBlock from '../../ui/CodeBlock.svelte';
	import { aggregateColumn, getFinalColumnOrder } from './datatable.js';
	import TableRow from './TableRow.svelte';
	import TotalRow from './TotalRow.svelte';
	import SubtotalRow from './SubtotalRow.svelte';
	import TableHeader from './TableHeader.svelte';
	import GroupRow from './GroupRow.svelte';
	import { ChevronsLeft, ChevronsRight, ChevronLeft, ChevronRight } from '@steeze-ui/tabler-icons';
	import EnterFullScreen from './EnterFullScreen.svelte';
	import Fullscreen from '../../../atoms/fullscreen/Fullscreen.svelte';
	import Column from './Column.svelte';
	import { Query } from '@evidence-dev/sdk/usql';
	import QueryLoad from '../../../atoms/query-load/QueryLoad.svelte';
	import { toasts } from '@evidence-dev/component-utilities/stores';
	import { query } from '@evidence-dev/universal-sql/client-duckdb';
	import Skeleton from '../../../atoms/skeletons/Skeleton.svelte';
	import { debounce } from 'perfect-debounce';

	// Set up props store
	let props = writable({});
	setContext(propKey, props);

	// Data, pagination, and row index numbers
	export let data;
	export let queryID = undefined;
	export let rows = 10; // number of rows to show
	$: rows = Number.parseInt(rows);

	export let rowNumbers = false;
	$: rowNumbers = rowNumbers === 'true' || rowNumbers === true;

	export let groupBy = undefined;
	export let groupsOpen = true; // starting toggle for groups - open or closed
	$: groupsOpen = groupsOpen === 'true' || groupsOpen === true;
	export let groupType = 'accordion'; // accordion | section
	export let accordionRowColor = undefined;
	export let groupNamePosition = 'middle'; // middle (default) | top | bottom

	if (groupType === 'section') {
		rowNumbers = false; // turn off row numbers
	}

	export let subtotals = false;
	$: subtotals = subtotals === 'true' || subtotals === true;

	export let subtotalRowColor = undefined;
	export let subtotalFontColor = undefined;

	let groupToggleStates = {};

	function handleToggle({ detail }) {
		const { groupName } = detail;
		groupToggleStates[groupName] = !groupToggleStates[groupName];
	}

	let paginated;
	$: paginated = data.length > rows && !groupBy;

	let hovering = false;

	let marginTop = '1.5em';
	let marginBottom = '1em';
	let paddingBottom = '0em';

	export let generateMarkdown = false;
	$: generateMarkdown = generateMarkdown === 'true' || generateMarkdown === true;

	// Table features
	export let search = false;
	$: search = search === 'true' || search === true;

	export let sortable = true;
	$: sortable = sortable === 'true' || sortable === true;

	export let downloadable = true;
	$: downloadable = downloadable === 'true' || downloadable === true;

	export let totalRow = false;
	$: totalRow = totalRow === 'true' || totalRow === true;

	export let totalRowColor = undefined;
	export let totalFontColor = undefined;

	export let isFullPage = false;

	// Row Links:
	export let link = undefined;

	export let showLinkCol = false; // hides link column when columns have not been explicitly selected
	$: showLinkCol = showLinkCol === 'true' || showLinkCol === true;

	let error = undefined;

	// ---------------------------------------------------------------------------------------
	// Add props to store to let child components access them
	// ---------------------------------------------------------------------------------------
	props.update((d) => {
		return { ...d, data, columns: [] };
	});

	// ---------------------------------------------------------------------------------------
	// STYLING
	// ---------------------------------------------------------------------------------------
	export let rowShading = false;
	$: rowShading = rowShading === 'true' || rowShading === true;

	export let rowLines = true;
	$: rowLines = rowLines === 'true' || rowLines === true;

	export let wrapTitles = false;
	$: wrapTitles = wrapTitles === 'true' || wrapTitles === true;

	export let headerColor = undefined;
	export let headerFontColor = 'var(--grey-900)';

	export let formatColumnTitles = true;
	$: formatColumnTitles = formatColumnTitles === 'true' || formatColumnTitles === true;

	export let backgroundColor = 'white';

	export let compact = undefined;

	// ---------------------------------------------------------------------------------------
	// DATA SETUP
	// ---------------------------------------------------------------------------------------

	let columnSummary;

	let priorityColumns = [groupBy];

	props.update((d) => {
		return { ...d, priorityColumns };
	});

	$: try {
		error = undefined;

		// CHECK INPUTS
		checkInputs(data);

		// GET COLUMN SUMMARY
		columnSummary = getColumnSummary(data, 'array');

		// PROCESS DATES
		// Filter for columns with type of "date"
		const dateCols = columnSummary
			.filter((d) => d.type === 'date' && !(data[0]?.[d.id] instanceof Date))
			.map((d) => d.id);

		for (let i = 0; i < dateCols.length; i++) {
			data = convertColumnToDate(data, dateCols[i]);
		}

		// Hide link column if columns have not been explicitly selected:
		for (let i = 0; i < columnSummary.length; i++) {
			columnSummary[i].show = showLinkCol === false && columnSummary[i].id === link ? false : true;
		}
	} catch (e) {
		error = e.message;
		if (strictBuild) {
			throw error;
		}
	}

	let index = 0;

	let inputPage = null;
	$: inputPageElWidth = `${(inputPage ?? 1).toString().length}ch`;

	// ---------------------------------------------------------------------------------------
	// SEARCH
	// ---------------------------------------------------------------------------------------
	let searchValue = '';
	/** @type {import("@evidence-dev/sdk/usql").QueryValue} */
	let filteredData;
	$: filteredData = data;
	let showNoResults = false;

	/** @type {ReturnValue<typeof Query["createReactive"]>}*/
	let searchFactory;
	$: if (Query.isQuery(data) && search) {
		searchFactory = debounce(
			Query.createReactive(
				{
					loadGracePeriod: 1000,
					callback: (v) => {
						filteredData = v;
					},
					execFn: query
				},
				data.opts,
				data
			),
			200
		);
	}

	$: if (searchFactory) {
		if (searchValue) {
			let searchCol =
				$props.columns.length > 0
					? $props.columns.map((c) => c.id)
					: data.columns.map((c) => c.column_name);
			searchFactory(
				data.search(
					searchValue,
					searchCol,
					searchValue.length === 1 ? 0.5 : searchValue.length >= 6 ? 0.9 : 0.8
				),
				data.opts
			);
		} else {
			searchFactory(data, data.opts);
		}
	}

	$: if (search && !Query.isQuery(data)) {
		toasts.add({
			status: 'warning',
			title: 'Search Failed',
			description: 'Please use a query instead.',
			timeout: 5000
		});
	}

	// ---------------------------------------------------------------------------------------
	// SORTING
	// ---------------------------------------------------------------------------------------

	let sortBy = { col: null, ascending: null };

	$: sort = (column) => {
		if (sortBy.col == column) {
			sortBy.ascending = !sortBy.ascending;
		} else {
			sortBy.col = column;
			sortBy.ascending = true;
		}

		// Modifier to sorting function for ascending or descending
		const sortModifier = sortBy.ascending ? 1 : -1;

		const forceTopOfAscending = (val) =>
			val === undefined || val === null || (typeof val === 'number' && isNaN(val));

		const sort = (a, b) =>
			(forceTopOfAscending(a[column]) && !forceTopOfAscending(b[column])) || a[column] < b[column]
				? -1 * sortModifier
				: (forceTopOfAscending(b[column]) && !forceTopOfAscending(a[column])) ||
					  a[column] > b[column]
					? 1 * sortModifier
					: 0;
		data.sort(sort);
		filteredData = filteredData.sort(sort);

		if (groupBy) {
			// sort within grouped data
			Object.keys(groupedData).forEach((groupName) => {
				groupedData[groupName] = groupedData[groupName].sort(sort);
			});
		}
	};

	let sortedGroupNames;
	$: if (groupBy && sortBy.col) {
		// Sorting groups based on aggregated values or group names
		sortedGroupNames = Object.entries(groupRowData)
			.sort((a, b) => {
				const valA = a[1][sortBy.col],
					valB = b[1][sortBy.col];
				// Use the existing sort logic but apply it to groupRowData's values
				if (
					(valA === undefined || valA === null || isNaN(valA)) &&
					valB !== undefined &&
					valB !== null &&
					!isNaN(valB)
				) {
					return -1 * (sortBy.ascending ? 1 : -1);
				}
				if (
					(valB === undefined || valB === null || isNaN(valB)) &&
					valA !== undefined &&
					valA !== null &&
					!isNaN(valA)
				) {
					return 1 * (sortBy.ascending ? 1 : -1);
				}
				if (valA < valB) {
					return -1 * (sortBy.ascending ? 1 : -1);
				} else if (valA > valB) {
					return 1 * (sortBy.ascending ? 1 : -1);
				}
				return 0;
			})
			.map((entry) => entry[0]); // Extract sorted group names
	} else {
		// Default to alphabetical order of group names or another criterion when not sorting by a specific column
		sortedGroupNames = Object.keys(groupedData).sort();
	}

	// Reset sort condition when data object is changed
	$: data, (sortBy = { col: null, ascending: null });

	// ---------------------------------------------------------------------------------------
	// PAGINATION
	// ---------------------------------------------------------------------------------------

	let totalRows;
	$: totalRows = filteredData.length;

	let displayedData = filteredData;

	let pageCount;
	let currentPage = 1;

	$: currentPage = Math.ceil((index + rows) / rows);
	$: currentPageElWidth = `${(currentPage ?? 1).toString().length}ch`;
	let max;

	$: goToPage = (pageNumber) => {
		index = pageNumber * rows;
		max = index + rows;
		currentPage = Math.ceil(max / rows);
		if (inputPage) {
			inputPage = Math.ceil(max / rows);
		}
		totalRows = filteredData.length;
		displayedData = filteredData.slice(index, index + rows);
	};

	let displayedPageLength = 0;

	$: if (paginated) {
		pageCount = Math.ceil(filteredData.length / rows);

		displayedData = filteredData.slice(index, index + rows);
		displayedPageLength = displayedData.length;
		if (pageCount < currentPage) {
			goToPage(pageCount - 1);
		} else if (currentPage < 1) {
			goToPage(0);
		}
	} else {
		currentPage = 1;
		displayedData = filteredData;
	}

	// ---------------------------------------------------------------------------------------
	// DATA FOR EXPORT
	// ---------------------------------------------------------------------------------------

	function dataSubset(data, selectedCols) {
		return data.map((obj) => {
			var toReturn = {}; //object that would give each desired key for each part in arr
			selectedCols.forEach((key) => (toReturn[key] = obj[key])); //placing wanted keys in toReturn
			return toReturn;
		});
	}

	let tableData;
	$: tableData =
		$props.columns.length > 0
			? dataSubset(
					data,
					$props.columns.map((d) => d.id)
				)
			: data;

	// ---------------------------------------------------------------------------------------
	// GROUPED DATA
	// ---------------------------------------------------------------------------------------

	let groupedData = {};
	let groupRowData = [];

	$: if (!error) {
		groupedData = data.reduce((acc, row) => {
			const groupName = row[groupBy];
			if (!acc[groupName]) {
				acc[groupName] = [];
			}
			acc[groupName].push(row);
			return acc;
		}, {});

		// After groupedData is populated, calculate aggregations for groupRowData
		groupRowData = Object.keys(groupedData).reduce((acc, groupName) => {
			acc[groupName] = {}; // Initialize groupRow object for this group

			// Get a list of columns to aggregate from $props.columns
			const columnsToAggregate = $props.columns.length > 0 ? $props.columns : columnSummary;

			columnsToAggregate.forEach((columnDef) => {
				const column = columnDef.id;
				const colType = columnSummary.find((d) => d.id === column)?.type;
				const totalAgg = columnDef.totalAgg;
				const weightCol = columnDef.weightCol;
				const rows = groupedData[groupName];
				acc[groupName][column] = aggregateColumn(rows, column, totalAgg, colType, weightCol);
			});

			return acc;
		}, {});

		// Update groupToggleStates only for new groups
		const existingGroups = Object.keys(groupToggleStates);
		Object.keys(groupedData).forEach((groupName) => {
			if (!existingGroups.includes(groupName)) {
				groupToggleStates[groupName] = groupsOpen; // Only add new groups with the default state
			}
			// Existing states are untouched
		});
	}

	let fullscreen = false;
	/** @type {number} */
	let innerHeight;
</script>

<svelte:window bind:innerHeight />

{#if !isFullPage && innerHeight !== undefined}
	<Fullscreen bind:open={fullscreen}>
		<!-- header and last row are 22.5+22.5 = 45px, middle rows are 23 -->
		{@const ROW_HEIGHT = 23}
		{@const Y_AXIS_PADDING = 45 + 234}
		<div class="pl-8 pt-4">
			<svelte:self
				{...$$props}
				rows={1 + Math.round((innerHeight - Y_AXIS_PADDING) / ROW_HEIGHT)}
				isFullPage
			>
				{#each $props.columns as column}
					<Column {...column} />
				{/each}
			</svelte:self>
		</div>
	</Fullscreen>
{/if}

{#if error === undefined}
	<slot />

	{#if link}
		<InvisibleLinks {data} {link} />
	{/if}
	{#each $props.columns.filter((column) => column.contentType === 'link') as column}
		<InvisibleLinks {data} link={column.id} />
	{/each}

	<div
		role="none"
		class="table-container"
		transition:slide|local
		style:margin-top={marginTop}
		style:margin-bottom={marginBottom}
		style:padding-bottom={paddingBottom}
		on:mouseenter={() => (hovering = true)}
		on:mouseleave={() => (hovering = false)}
	>
		{#if search}
			<SearchBar bind:value={searchValue} searchFunction={() => {}} />
		{/if}

		<div class="scrollbox" style:background-color={backgroundColor}>
			<table>
				<TableHeader
					{rowNumbers}
					{headerColor}
					{headerFontColor}
					finalColumnOrder={getFinalColumnOrder(
						$props.columns.length > 0 ? $props.columns.map((d) => d.id) : Object.keys(data[0]),
						$props.priorityColumns
					)}
					{columnSummary}
					{compact}
					{sortable}
					{sort}
					{formatColumnTitles}
					{sortBy}
					{wrapTitles}
				/>

				<QueryLoad data={filteredData}>
					<svelte:fragment slot="skeleton">
						<tr>
							<td colspan={filteredData.columns.length} class="h-32">
								<Skeleton />
							</td>
						</tr>
					</svelte:fragment>
					{#if groupBy && groupedData && searchValue === ''}
						{#each sortedGroupNames as groupName}
							{#if groupType === 'accordion'}
								<GroupRow
									{groupName}
									currentGroupData={groupedData[groupName]}
									toggled={groupToggleStates[groupName]}
									on:toggle={handleToggle}
									{columnSummary}
									rowColor={accordionRowColor}
									{rowNumbers}
									{subtotals}
									{compact}
									finalColumnOrder={getFinalColumnOrder(
										$props.columns.length > 0
											? $props.columns.map((d) => d.id)
											: Object.keys(data[0]),
										$props.priorityColumns
									)}
								/>
								{#if groupToggleStates[groupName]}
									<TableRow
										displayedData={groupedData[groupName]}
										{groupType}
										{rowShading}
										{link}
										{rowNumbers}
										{rowLines}
										{compact}
										{index}
										{columnSummary}
										grouped={true}
										groupColumn={groupBy}
										finalColumnOrder={getFinalColumnOrder(
											$props.columns.length > 0
												? $props.columns.map((d) => d.id)
												: Object.keys(data[0]),
											$props.priorityColumns
										)}
									/>
								{/if}
							{:else if groupType === 'section'}
								<TableRow
									groupColumn={groupBy}
									{groupType}
									rowSpan={groupedData[groupName].length}
									displayedData={groupedData[groupName]}
									{rowShading}
									{link}
									{rowNumbers}
									{rowLines}
									{compact}
									{index}
									{columnSummary}
									grouped={true}
									{groupNamePosition}
									finalColumnOrder={getFinalColumnOrder(
										$props.columns.length > 0
											? $props.columns.map((d) => d.id)
											: Object.keys(data[0]),
										$props.priorityColumns
									)}
								/>
								{#if subtotals}
									<SubtotalRow
										{groupName}
										currentGroupData={groupedData[groupName]}
										{columnSummary}
										rowColor={subtotalRowColor}
										fontColor={subtotalFontColor}
										{groupType}
										{groupBy}
										{compact}
										finalColumnOrder={getFinalColumnOrder(
											$props.columns.length > 0
												? $props.columns.map((d) => d.id)
												: Object.keys(data[0]),
											$props.priorityColumns
										)}
									/>
								{/if}
							{/if}
						{/each}
					{:else}
						<TableRow
							{displayedData}
							{rowShading}
							{link}
							{rowNumbers}
							{rowLines}
							{compact}
							{index}
							{columnSummary}
							finalColumnOrder={getFinalColumnOrder(
								$props.columns.length > 0 ? $props.columns.map((d) => d.id) : Object.keys(data[0]),
								$props.priorityColumns
							)}
						/>
					{/if}

					{#if totalRow && searchValue === ''}
						<TotalRow
							{data}
							{rowNumbers}
							{columnSummary}
							rowColor={totalRowColor}
							fontColor={totalFontColor}
							{groupType}
							{compact}
							finalColumnOrder={getFinalColumnOrder(
								$props.columns.length > 0 ? $props.columns.map((d) => d.id) : Object.keys(data[0]),
								$props.priorityColumns
							)}
						/>
					{/if}
				</QueryLoad>
			</table>
		</div>

		<div class="noresults" class:shownoresults={showNoResults}>No Results</div>

		{#if paginated && pageCount > 1}
			<div class="pagination">
				<div class="page-labels mr-auto">
					<button
						aria-label="first-page"
						class="page-changer"
						class:hovering
						disabled={currentPage === 1}
						on:click={() => goToPage(0)}
					>
						<div class="page-icon flex items-center">
							<Icon src={ChevronsLeft} />
						</div>
					</button>
					<button
						aria-label="previous-page"
						class="page-changer"
						class:hovering
						disabled={currentPage === 1}
						on:click={() => goToPage(currentPage - 2)}
					>
						<div class="page-icon h-[0.83em] flex items-center">
							<Icon src={ChevronLeft} class="h-[0.83em]" />
						</div>
					</button>
					<span class="page-count">
						Page
						<input
							class="page-input"
							class:hovering
							class:error={inputPage > pageCount}
							style="width: {inputPage ? inputPageElWidth : currentPageElWidth};"
							type="number"
							bind:value={inputPage}
							on:keyup={() => goToPage((inputPage ?? 1) - 1)}
							on:change={() => goToPage((inputPage ?? 1) - 1)}
							placeholder={currentPage}
						/>
						/
						<span class="page-count ml-1">{pageCount.toLocaleString()}</span>
					</span>
					<span class="print-page-count">
						{displayedPageLength.toLocaleString()} of {totalRows.toLocaleString()} records
					</span>
					<button
						aria-label="next-page"
						class="page-changer"
						class:hovering
						disabled={currentPage === pageCount}
						on:click={() => goToPage(currentPage)}
					>
						<div class="page-icon h-[0.83em] flex items-center">
							<Icon src={ChevronRight} class="h-[0.83em]" />
						</div>
					</button>
					<button
						aria-label="last-page"
						class="page-changer"
						class:hovering
						disabled={currentPage === pageCount}
						on:click={() => goToPage(pageCount - 1)}
					>
						<div class="page-icon flex items-center">
							<Icon src={ChevronsRight} />
						</div>
					</button>
				</div>
				{#if downloadable}
					<DownloadData class="download-button" data={tableData} {queryID} display={hovering} />
				{/if}
				{#if !isFullPage}
					<EnterFullScreen on:click={() => (fullscreen = true)} display={hovering} />
				{/if}
			</div>
		{:else}
			<div class="table-footer">
				{#if downloadable}
					<DownloadData class="download-button" data={tableData} {queryID} display={hovering} />
				{/if}
				{#if !isFullPage}
					<EnterFullScreen on:click={() => (fullscreen = true)} display={hovering} />
				{/if}
			</div>
		{/if}
	</div>

	{#if generateMarkdown}
		{#if queryID}
			<CodeBlock>
				{`<DataTable data={${queryID}}>`}
				<br />
				{#each Object.keys(data[0]) as column}
					{`   <Column id=${column.includes(' ') ? `'${column}'` : column}/>`}
					<br />
				{/each}
				{`</DataTable>`}
			</CodeBlock>
		{/if}
	{/if}
{:else}
	<ErrorChart {error} chartType="Data Table" />
{/if}

<style>
	.table-container {
		font-size: 9.5pt;
	}

	.scrollbox {
		width: 100%;
		overflow-x: auto;
		/* border-bottom: 1px solid var(--grey-200);    */
		scrollbar-width: thin;
		scrollbar-color: var(--scrollbar-color) var(--scrollbar-track-color);
	}

	:root {
		--scrollbar-track-color: transparent;
		--scrollbar-color: rgba(0, 0, 0, 0.2);
		--scrollbar-active-color: rgba(0, 0, 0, 0.4);
		--scrollbar-size: 0.75rem;
		--scrollbar-minlength: 1.5rem; /* Minimum length of scrollbar thumb (width of horizontal, height of vertical) */
	}

	.scrollbox::-webkit-scrollbar {
		height: var(--scrollbar-size);
		width: var(--scrollbar-size);
	}

	.scrollbox::-webkit-scrollbar-track {
		background-color: var(--scrollbar-track-color);
	}

	.scrollbox::-webkit-scrollbar-thumb {
		background-color: var(--scrollbar-color);
		border-radius: 7px;
		background-clip: padding-box;
	}

	.scrollbox::-webkit-scrollbar-thumb:hover {
		background-color: var(--scrollbar-active-color);
	}

	.scrollbox::-webkit-scrollbar-thumb:vertical {
		min-height: var(--scrollbar-minlength);
		border: 3px solid transparent;
	}

	.scrollbox::-webkit-scrollbar-thumb:horizontal {
		min-width: var(--scrollbar-minlength);
		border: 3px solid transparent;
	}

	table {
		display: table;
		width: 100%;
		border-collapse: collapse;
		font-variant-numeric: tabular-nums;
	}

	.page-changer {
		padding: 0;
		color: var(--grey-400);
		height: 1.1em;
		width: 1.1em;
	}

	.pagination {
		font-size: 12px;
		display: flex;
		align-items: center;
		justify-content: flex-end;
		height: 2em;
		font-family: var(--ui-font-family);
		color: var(--grey-500);
		-webkit-user-select: none;
		-moz-user-select: none;
		user-select: none;
		text-align: right;
		margin-top: 0.5em;
		margin-bottom: 1.8em;
		font-variant-numeric: tabular-nums;
	}

	.page-labels {
		display: flex;
		justify-content: flex-start;
		align-items: center;
		gap: 3px;
	}

	.page-changer {
		font-size: 20px;
		background: none;
		border: none;
		cursor: pointer;
		transition: color 200ms;
	}

	.page-changer.hovering {
		color: var(--blue-600);
		transition: color 200ms;
	}

	.page-changer:disabled {
		cursor: auto;
		color: var(--grey-300);
		-webkit-user-select: none;
		-moz-user-select: none;
		user-select: none;
		transition: color 200ms;
	}

	.page-icon {
		height: 1em;
		width: 1em;
	}

	.page-input {
		box-sizing: content-box;
		text-align: center;
		padding: 0.25em 0.5em;
		margin: 0;
		border: 1px solid transparent;
		border-radius: 4px;
		font-size: 12px;
		color: var(--grey-500);
	}

	.table-footer {
		display: flex;
		justify-content: flex-end;
		align-items: center;
		margin: 10px 0px;
		font-size: 12px;
		height: 9px;
	}

	/* Remove number buttons in input box*/
	.page-input::-webkit-outer-spin-button,
	.page-input::-webkit-inner-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}

	/* Firefox */
	.page-input[type='number'] {
		-moz-appearance: textfield;
		-webkit-appearance: textfield;
		appearance: textfield;
	}

	.page-input.hovering {
		border: 1px solid var(--grey-200);
	}

	.page-input.error {
		border: 1px solid var(--red-600);
	}

	.page-input::-moz-placeholder {
		color: var(--grey-500);
	}

	.page-input::placeholder {
		color: var(--grey-500);
	}

	button:enabled > .page-icon:hover {
		color: var(--blue-800);
	}

	*:focus {
		outline: none;
	}

	::-moz-placeholder {
		/* Chrome, Firefox, Opera, Safari 10.1+ */
		color: var(--grey-400);
		opacity: 1; /* Firefox */
	}

	::placeholder {
		/* Chrome, Firefox, Opera, Safari 10.1+ */
		color: var(--grey-400);
		opacity: 1; /* Firefox */
	}

	:-ms-input-placeholder {
		/* Internet Explorer 10-11 */
		color: var(--grey-400);
	}

	::-ms-input-placeholder {
		/* Microsoft Edge */
		color: var(--grey-400);
	}

	.noresults {
		display: none;
		color: var(--grey-400);
		text-align: center;
		margin-top: 5px;
	}

	.shownoresults {
		display: block;
	}

	.print-page-count {
		display: none;
	}

	@media (max-width: 600px) {
		.page-changer {
			height: 1.2em;
			width: 1.2em;
		}

		.page-icon {
			height: 1.2em;
			width: 1.2em;
		}

		.page-count {
			font-size: 1.1em;
		}

		.page-input {
			font-size: 1.1em;
		}
	}

	@media print {
		.pagination {
			-moz-column-break-inside: avoid;
			break-inside: avoid;
		}

		.page-changer {
			display: none;
		}

		.page-count {
			display: none;
		}

		.print-page-count {
			display: inline;
		}
	}
</style>
