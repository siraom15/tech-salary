<script lang="ts">
	import { newestEntry } from '$lib/stores/salaryStore';
	import type { CompanySalary } from '$lib/types/salary';

	export let salaryData: CompanySalary[];

	const itemsPerPage = 5;
	let currentPage = 1;
	let searchQuery = '';

	const formatCurrency = (amount: string | number | null | undefined): string => {
		if (amount == null || amount === 'N/A') return '-';
		if (typeof amount === 'string') {
			const numAmount = parseFloat(amount);
			if (isNaN(numAmount)) return '-';
			amount = numAmount;
		}
		return amount.toLocaleString('th-TH', {
			style: 'currency',
			currency: 'THB',
			minimumFractionDigits: 0,
			maximumFractionDigits: 0
		});
	};

	function isNewEntry(company: string, position: string): boolean {
		return $newestEntry === `${company}-${position}`;
	}

	// Filter data based on search query
	$: filteredData = searchQuery
		? salaryData.filter((item) => {
				const searchLower = searchQuery.toLowerCase();
				return (
					item.company.toLowerCase().includes(searchLower) ||
					item.level.toLowerCase().includes(searchLower) ||
					item.tag.toLowerCase().includes(searchLower) ||
					String(item.experience).includes(searchQuery)
				);
		  })
		: salaryData;

	$: sortedSalaryData = [...filteredData].sort((a, b) => {
		const companyCompare = a.company.localeCompare(b.company);
		if (companyCompare !== 0) return companyCompare;

		const totalA = (Number(a.salary) || 0) + (Number(a.stock) || 0) + (Number(a.bonus) || 0);
		const totalB = (Number(b.salary) || 0) + (Number(b.stock) || 0) + (Number(b.bonus) || 0);
		return totalB - totalA;
	});

	// Reset to first page when search query changes
	$: {
		if (searchQuery) {
			currentPage = 1;
		}
	}

	$: totalPages = Math.ceil(sortedSalaryData.length / itemsPerPage);
	$: paginatedData = sortedSalaryData.slice(
		(currentPage - 1) * itemsPerPage,
		currentPage * itemsPerPage
	);

	function nextPage() {
		if (currentPage < totalPages) currentPage++;
	}

	function prevPage() {
		if (currentPage > 1) currentPage--;
	}

	function generatePageNumbers(
		current: number,
		total: number,
		isMobile: boolean
	): (number | string)[] {
		const pages: (number | string)[] = [];
		const delta = isMobile ? 1 : 2;

		for (let i = 1; i <= total; i++) {
			if (i === 1 || i === total || (i >= current - delta && i <= current + delta)) {
				pages.push(i);
			} else if ((i === current - delta - 1 && i > 1) || (i === current + delta + 1 && i < total)) {
				pages.push('...');
			}
		}

		return [...new Set(pages)];
	}

	let isMobile: boolean;

	function checkMobile() {
		isMobile = window.innerWidth < 640;
	}

	function goToPage(page: number) {
		currentPage = page;
	}

	function clearSearch() {
		searchQuery = '';
	}
</script>

<svelte:window on:resize={checkMobile} />

<div class="container mx-auto p-2 sm:p-4">
	<!-- Search input -->
	<div class="mb-4">
		<div class="relative">
			<input
				type="text"
				bind:value={searchQuery}
				placeholder="Search by company, level, tag..."
				class="w-full rounded-lg border border-gray-300 px-4 py-2 pr-10 text-sm focus:border-[#0056a9] focus:outline-none focus:ring-2 focus:ring-[#0056a9]/20 sm:text-base"
			/>
			{#if searchQuery}
				<button
					on:click={clearSearch}
					class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-500 hover:text-gray-700"
					aria-label="Clear search"
				>
					✕
				</button>
			{:else}
				<div class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400">
					<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
					</svg>
				</div>
			{/if}
		</div>
	</div>

	{#if sortedSalaryData.length === 0}
		<div class="rounded-xl border border-gray-100 bg-white p-8 text-center shadow-lg">
			<p class="text-gray-600">No results found for "{searchQuery}". Try a different search term.</p>
		</div>
	{:else}
		<div class="overflow-x-auto rounded-xl border border-gray-100 shadow-lg">
			<table class="min-w-full bg-white text-sm sm:text-base">
				<thead>
					<tr class="bg-gradient-to-r from-[#0056a9] to-[#0066cc] text-white">
						<th
							class="px-3 py-3 text-left text-xs font-semibold tracking-wider sm:px-6 sm:py-4 sm:text-sm"
							>Company</th
						>
						<th
							class="px-3 py-3 text-left text-xs font-semibold tracking-wider sm:px-6 sm:py-4 sm:text-sm"
							>Level</th
						>
						<th
							class="px-3 py-3 text-left text-xs font-semibold tracking-wider sm:px-6 sm:py-4 sm:text-sm"
							>Tag</th
						>
						<th
							class="px-3 py-3 text-left text-xs font-semibold tracking-wider sm:px-6 sm:py-4 sm:text-sm"
							>Experience</th
						>
						<th
							class="px-3 py-3 text-left text-xs font-semibold tracking-wider sm:px-6 sm:py-4 sm:text-sm"
							>Total Compensation</th
						>
						<th
							class="px-3 py-3 text-right text-xs font-semibold tracking-wider sm:px-6 sm:py-4 sm:text-sm"
							>Base Salary</th
						>
						<th
							class="hidden px-3 py-3 text-right text-xs font-semibold tracking-wider sm:table-cell sm:px-6 sm:py-4 sm:text-sm"
							>Monthly Stock</th
						>
						<th
							class="hidden px-3 py-3 text-right text-xs font-semibold tracking-wider sm:table-cell sm:px-6 sm:py-4 sm:text-sm"
							>Bonus</th
						>
					</tr>
				</thead>
				<tbody class="divide-y divide-gray-100">
					{#each paginatedData as { company, level, tag, experience, totalCompensation, salary, stock, bonus }, i}
						<tr
							class="transition-colors duration-200 {i % 2 === 0
								? 'bg-white'
								: 'bg-slate-50/50'} {isNewEntry(company, level)
								? 'animate-pulse bg-green-50'
								: 'hover:bg-blue-50/50'}"
						>
							<td class="px-3 py-3 text-xs font-medium text-gray-900 sm:px-6 sm:py-4 sm:text-sm"
								>{company}</td
							>
							<td class="px-3 py-3 text-xs text-gray-700 sm:px-6 sm:py-4 sm:text-sm">{level}</td>
							<td class="px-3 py-3 text-xs text-gray-700 sm:px-6 sm:py-4 sm:text-sm">{tag}</td>
							<td class="px-3 py-3 text-xs text-gray-700 sm:px-6 sm:py-4 sm:text-sm"
								>{experience} Years</td
							>
							<td
								class="px-3 py-3 text-right text-xs font-medium text-gray-900 sm:px-6 sm:py-4 sm:text-sm"
								>{formatCurrency(totalCompensation)}</td
							>
							<td
								class="px-3 py-3 text-right text-xs font-medium text-gray-900 sm:px-6 sm:py-4 sm:text-sm"
								>{formatCurrency(salary)}</td
							>
							<td
								class="hidden px-3 py-3 text-right text-xs font-medium text-gray-900 sm:table-cell sm:px-6 sm:py-4 sm:text-sm"
								>{formatCurrency(stock)}</td
							>
							<td
								class="hidden px-3 py-3 text-right text-xs font-medium text-gray-900 sm:table-cell sm:px-6 sm:py-4 sm:text-sm"
								>{formatCurrency(bonus)}</td
							>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	{/if}

	<div
		class="mt-4 flex flex-col items-center justify-between gap-2 px-2 sm:mt-6 sm:flex-row sm:gap-4 sm:px-4"
	>
		<div class="text-xs text-gray-700 sm:text-sm">
			{#if sortedSalaryData.length > 0}
				Showing {(currentPage - 1) * itemsPerPage + 1} to {Math.min(
					currentPage * itemsPerPage,
					sortedSalaryData.length
				)} of {sortedSalaryData.length} entries
				{#if searchQuery}
					(filtered from {salaryData.length} total entries)
				{/if}
			{:else}
				No matching entries found
			{/if}
		</div>
		{#if totalPages > 0}
			<div class="flex items-center gap-1 sm:gap-2">
				<button
					on:click={prevPage}
					disabled={currentPage === 1}
					class="rounded-lg border px-2 py-1 text-xs font-medium transition-colors duration-200 sm:px-4 sm:py-2 sm:text-sm {currentPage ===
					1
						? 'cursor-not-allowed border-gray-200 bg-gray-50 text-gray-400'
						: 'border-[#0056a9] bg-white text-[#0056a9] hover:bg-[#0056a9] hover:text-white'}"
					aria-label="Previous page"
				>
					←
				</button>

				{#each generatePageNumbers(currentPage, totalPages, isMobile) as page}
					{#if typeof page === 'string'}
						<span class="px-2 py-1 text-xs text-gray-700 sm:px-4 sm:py-2 sm:text-sm">{page}</span>
					{:else}
						<button
							on:click={() => goToPage(page)}
							class="rounded-lg border px-2 py-1 text-xs font-medium transition-colors duration-200 sm:px-4 sm:py-2 sm:text-sm {currentPage ===
							page
								? 'border-[#0056a9] bg-[#0056a9] text-white'
								: 'border-[#0056a9] bg-white text-[#0056a9] hover:bg-[#0056a9] hover:text-white'}"
						>
							{page}
						</button>
					{/if}
				{/each}

				<button
					on:click={nextPage}
					disabled={currentPage === totalPages}
					class="rounded-lg border px-2 py-1 text-xs font-medium transition-colors duration-200 sm:px-4 sm:py-2 sm:text-sm {currentPage ===
					totalPages
						? 'cursor-not-allowed border-gray-200 bg-gray-50 text-gray-400'
						: 'border-[#0056a9] bg-white text-[#0056a9] hover:bg-[#0056a9] hover:text-white'}"
					aria-label="Next page"
				>
					→
				</button>
			</div>
		{/if}
	</div>
</div>