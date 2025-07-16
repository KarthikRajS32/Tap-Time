<script lang="ts">
    import { onMount } from 'svelte';
    import jsPDF from 'jspdf';
    // Import autoTable like this:
    import autoTable from 'jspdf-autotable';

    const apiUrlBase = 'https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/report/dateRangeReportGet';

    // State
    let startDateHeader = '';
    let endDateHeader = '';
    let reportName = '';
    export let reportTypeHeading = '';
    // @ts-ignore
    let employees = [];
    let isLoading = true;
    let searchTerm = '';
    let sortConfig = { key: 'name', direction: 'asc' };
    let currentPage = 1;
    let itemsPerPage = 10;

    // Computed properties
    // @ts-ignore
    $: filteredEmployees = employees
        .filter(emp => 
            emp.name.toLowerCase().includes(searchTerm.toLowerCase()) || 
            emp.pin.toLowerCase().includes(searchTerm.toLowerCase()) ||
            emp.hoursWorked.toLowerCase().includes(searchTerm.toLowerCase())
        )
        .sort((a, b) => {
            if (a[sortConfig.key] < b[sortConfig.key]) {
                return sortConfig.direction === 'asc' ? -1 : 1;
            }
            if (a[sortConfig.key] > b[sortConfig.key]) {
                return sortConfig.direction === 'asc' ? 1 : -1;
            }
            return 0;
        });

    $: paginatedEmployees = filteredEmployees.slice(
        (currentPage - 1) * itemsPerPage,
        currentPage * itemsPerPage
    );

    $: totalPages = Math.ceil(filteredEmployees.length / itemsPerPage);

    onMount(() => {
        const selectedValue = localStorage.getItem('reportType');
        reportName = `${selectedValue} Report`;
        reportTypeHeading = `${selectedValue} Report`;
        loadReportData();
    });

    // @ts-ignore
    function requestSort(key) {
        let direction = 'asc';
        if (sortConfig.key === key && sortConfig.direction === 'asc') {
            direction = 'desc';
        }
        sortConfig = { key, direction };
        currentPage = 1; // Reset to first page when sorting
    }

    async function loadReportData() {
        isLoading = true;
        const cid = localStorage.getItem('companyID');
        const dateRange = getDateRange();
        
        startDateHeader = dateRange.startRange;
        endDateHeader = dateRange.endRange;

        try {
            const response = await fetch(`${apiUrlBase}/${cid}/${startDateHeader}/${endDateHeader}`);
            if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
            
            const data = await response.json();
            employees = Object.entries(calculateTotalTimeWorked(data))
                .map(([pin, empData]) => {
                    const { name, totalHoursWorked } = empData as { name: string; totalHoursWorked: string };
                    return {
                        pin,
                        name,
                        hoursWorked: totalHoursWorked
                    };
                });
        } catch (error) {
            console.error('Failed to load report:', error);
            employees = [];
        } finally {
            isLoading = false;
        }
    }

    function getDateRange() {
        const selectedValue = localStorage.getItem('reportType');
        
        switch (selectedValue) {
            case "Weekly": return getLastWeekDateRange();
            case "Monthly": return getLastMonthStartAndEndDates();
            case "Bimonthly": return getLastMonthStartAndEndDates();
            case "Biweekly": return getLastTwoWeeksDateRange();
            default: return { startRange: '', endRange: '' };
        }
    }

    function getLastWeekDateRange() {
        const now = new Date();
        const dayOfWeek = now.getDay(); // 0 (Sun) - 6 (Sat)
        const lastSunday = new Date(now);
        lastSunday.setDate(now.getDate() - dayOfWeek - 7);
        const lastSaturday = new Date(now);
        lastSaturday.setDate(now.getDate() - dayOfWeek - 1);
        return {
            startRange: formatDate(lastSunday),
            endRange: formatDate(lastSaturday)
        };
    }

    function getLastMonthStartAndEndDates() {
        const now = new Date();
        const start = new Date(now.getFullYear(), now.getMonth() - 1, 1);
        const end = new Date(now.getFullYear(), now.getMonth(), 0);
        return {
            startRange: formatDate(start),
            endRange: formatDate(end)
        };
    }

    function getLastTwoWeeksDateRange() {
        const now = new Date();
        const dayOfWeek = now.getDay(); // 0 (Sun) - 6 (Sat)
        // End date is last Saturday
        const endDate = new Date(now);
        endDate.setDate(now.getDate() - dayOfWeek - 1);
        // Start date is two Sundays before (13 days before end date)
        const startDate = new Date(endDate);
        startDate.setDate(endDate.getDate() - 13);
        return {
            startRange: formatDate(startDate),
            endRange: formatDate(endDate)
        };
    }

    // @ts-ignore
    function formatDate(date) {
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, '0');
        const day = String(date.getDate()).padStart(2, '0');
        return `${year}-${month}-${day}`;
    }

    // @ts-ignore
    function calculateTotalTimeWorked(data) {
        const employeeTimes = {};

        // @ts-ignore
        data.forEach(entry => {
            const { Name, Pin, CheckInTime, CheckOutTime } = entry;

            if (!Name || !Pin || !CheckInTime) return;

            const checkInDate = new Date(CheckInTime);
            const checkOutDate = CheckOutTime ? new Date(CheckOutTime) : new Date();
            // @ts-ignore
            const timeDifferenceInMinutes = Math.floor((checkOutDate - checkInDate) / 1000 / 60);

            // @ts-ignore
            if (!employeeTimes[Pin]) {
                // @ts-ignore
                employeeTimes[Pin] = { name: Name, totalMinutes: 0 };
            }

            // @ts-ignore
            employeeTimes[Pin].totalMinutes += timeDifferenceInMinutes;
        });

        for (const [pin, details] of Object.entries(employeeTimes) as [string, { name: string; totalMinutes: number; totalHoursWorked?: string }][]) {
            details.totalHoursWorked = minutesToTime(details.totalMinutes);
        }
        
        return employeeTimes;
    }

    // @ts-ignore
    function minutesToTime(minutes) {
        const hours = Math.floor(minutes / 60);
        const mins = minutes % 60;
        return `${hours}:${mins.toString().padStart(2, '0')}`;
    }

    function downloadPDF() {
    if (filteredEmployees.length === 0) {
        alert('No data to download');
        return;
    }

    const doc = new jsPDF();
    const formattedDateRange = `${startDateHeader} to ${endDateHeader}`;
    
    doc.setFontSize(14);
    doc.text(`${reportTypeHeading} (${formattedDateRange})`, 14, 10);

    const tableColumn = ["Name", "Pin", "Time Worked Hours (HH:MM)"];
    const tableRows = filteredEmployees.map(emp => [
        emp.name || '--',
        emp.pin || '--',
        emp.hoursWorked || '--'
    ]);

    autoTable(doc, {
        head: [tableColumn],
        body: tableRows,
        startY: 20,
        headStyles: {
            
            fillColor: [2, 6, 111],
            textColor: 255,
            fontSize: 10,
            fontStyle: 'bold',
        },
        styles: { fontSize: 10 },
        theme: 'grid'
    });

    const fileName = `${reportTypeHeading.toLowerCase().replace(/\s+/g, '_')}.pdf`;
    doc.save(fileName);
}

    function downloadCSV() {
        if (filteredEmployees.length === 0) {
            alert('No data to download');
            return;
        }

        const headers = ['Employee Name', 'Employee ID', 'Total Worked Hours (HH:MM)'];
        const csvData = filteredEmployees.map(employee => [
            employee.name,
            employee.pin,
            employee.hoursWorked
        ]);

        let csvContent = headers.join(',') + '\n';
        csvData.forEach(row => {
            csvContent += row.join(',') + '\n';
        });

        const blob = new Blob([csvContent], { type: 'text/csv' });
        const url = window.URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `${reportName.toLowerCase().replace(' ', '_')}_${startDateHeader}_to_${endDateHeader}.csv`;
        a.click();
        window.URL.revokeObjectURL(url);
    }
</script>

<div class="bg-gray-100">
<div class="pt-16 md:pt-18 sm:pt-2">

    {#if isLoading}
        <div class="fixed inset-0 flex items-center justify-center z-50"    
        style="background: rgba(0, 0, 0, 0.5)">
        <div class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"></div>
        </div>
    {/if}

    <!-- Navigation -->
    <nav class="bg-white shadow">
        <div class="max-w-8xl mx-auto px-4 sm:px-6 lg:px-8">
          <div class="flex flex-col md:flex-row justify-between items-center h-auto md:h-16 py-4 md:py-0 justify-end">
            
            <!-- Menu Links - visible on all devices -->
            <div class="flex flex-col md:flex-row space-y-2 md:space-y-0 md:space-x-4 text-center w-auto md:w-auto">
              <a href="/reportsummary" class="px-4 py-2 text-[#02066F]  font-semibold rounded-full">Today's Report</a>
              <a href="/daywisereport" class="px-4 py-2 text-[#02066F] font-semibold rounded-full">Day Wise Report</a>
              <a href="/salariedreport" class="px-4 py-2 bg-[#02066F] text-white font-semibold rounded-full">{reportTypeHeading}</a>
            </div>
          </div>
        </div>
    </nav>
   
    <!-- Main Content -->
    <main class="container mx-auto px-4 py-8">
        <h1 class="text-2xl font-bold text-center mb-8">{reportTypeHeading}</h1>

        <!-- Date Range Display -->
        <div class="flex flex-col max-w-5xl mx-auto md:flex-row justify-between mb-6 p-4 rounded-lg">
            <div class="mb-2 md:mb-0">
                <span class="text-md md:text-lg font-semibold text-gray-800">Start Date: </span>
                <span class="text-md md:text-lg font-semibold text-gray-800">{startDateHeader}</span>
            </div>
            <div>
                <span class="text-md md:text-lg font-semibold text-gray-800">End Date: </span>
                <span class="text-md md:text-lg font-semibold text-gray-800">{endDateHeader}</span>
            </div>
        </div>

        <!-- Search and Controls -->
        <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-sm overflow-hidden mb-8 border-1 border-gray-300">
            <div class="p-4 sm:p-6 overflow-x-auto">
                <!-- Download Buttons -->
                <!-- {#if filteredEmployees.length > 0} -->
                    <div class="flex flex-col gap-2 sm:flex-row justify-evenly items-center mb-6">
                        <button 
                            on:click={downloadPDF}
                            class="text-[#02066F] hover:text-black px-4 py-2 rounded-lg transition-colors cursor-pointer border-1 border-[#02066F]"
                        >
                            Download PDF
                        </button>
                        <button 
                            on:click={downloadCSV}
                            class="text-[#02066F] hover:text-black px-4 py-2 rounded-lg transition-colors cursor-pointer border-1 border-[#02066F]"
                        >
                            Download CSV
                        </button>
                    </div>
                <!-- {/if} -->

                <!-- Search and Entries Selector -->
                <div class="flex flex-col sm:flex-row justify-between items-center gap-4 mb-4">
                    <div class="flex items-center gap-2">
                        <label for="entries" class="text-base font-semibold text-gray-700">Show:</label>
                        <select 
                            id="entries"
                            bind:value={itemsPerPage}
                            class="border border-gray-400 rounded-md px-2 py-1 text-sm focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                        >
                            <option value="10">10</option>
                            <option value="25">25</option>
                            <option value="50">50</option>
                            <option value="100">100</option>
                        </select>
                        <span class="text-base font-semibold text-gray-700">entries</span>
                    </div>
                    
                    <div class="mb-4 flex flex-col sm:flex-row sm:items-center sm:justify-end gap-2">
                        <label for="search" class="text-base font-semibold text-gray-800">Search:</label>
                        <input
                            id="search"
                            type="text"
                            bind:value={searchTerm}
                            placeholder=""
                            class="w-full sm:w-64 px-2 py-1 border border-gray-500 rounded-md focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                        />
                    </div>
                </div>
           
                <!-- Employee Table -->
                <div class="overflow-hidden">
                    <!-- {#if filteredEmployees.length <= 0}
                        <div class="text-center p-8 text-gray-700">
                            {searchTerm ? 'No matching employees found' : 'No records available'}
                        </div>
                    {/if} -->
                    <!-- {:else} -->
                        <div class="overflow-x-auto w-full max-w-full sm:rounded-lg">
                            <table class="min-w-[600px] sm:min-w-full border border-gray-300 text-sm">
                                <thead class="bg-[#02066F] text-white">
                                    <tr>
                                        <th 
                                            class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer border-r" 
                                            on:click={() => requestSort('name')}
                                        >
                                        <div class="flex items-center justify-center">
                                            Name
                                            {#if sortConfig.key === 'name'}
                                                <span class="ml-6 text-lg">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                            {:else}
                                                <span class="ml-6 text-lg">↑↓</span>
                                            {/if}
                                            </div>
                                        </th>
                                        <th 
                                            class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer border-r"
                                            on:click={() => requestSort('pin')}
                                        >
                                         <div class="flex items-center justify-center">
                                            Pin
                                            {#if sortConfig.key === 'pin'}
                                                <span class="ml-1 text-lg">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                            {:else}
                                               <span class="ml-6 text-lg">↑↓</span>
                                            {/if}
                                            </div>
                                        </th>
                                        <th 
                                            class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer border-r"
                                            on:click={() => requestSort('hoursWorked')}
                                        >
                                        <div class="flex items-center justify-center">
                                           Total Worked Hours (HH:MM)
                                            {#if sortConfig.key === 'hoursWorked'}
                                                <span class="ml-1 text-lg">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                            {:else}
                                               <span class="ml-6 text-lg">↑↓</span>
                                            {/if}
                                            </div>
                                        </th>
                                    </tr>
                                </thead>
                                <tbody class="bg-white divide-y divide-gray-200">
                                    {#if filteredEmployees.length === 0}
                                        <!-- <div class="px-4 py-4 text-center items-center justify-center text-gray-500"> -->
                                        <tr>    
                                            <td colspan="5" class="px-4 py-4 text-center text-gray-500">     
                                                No matching records found
                                            </td>
                                        </tr>
                                        <!-- </div> -->
                                    {:else}
                                        {#each paginatedEmployees as employee}
                                        <tr class="hover:bg-gray-50 text-center">
                                            <td class="px-6 py-3 whitespace-nowrap text-sm font-semibold text-gray-900 border-gray-300 border-r">{employee.name}</td>
                                            <td class="px-6 py-3 whitespace-nowrap text-sm font-semibold text-gray-900 border-gray-300 border-r">{employee.pin}</td>
                                            <td class="px-6 py-3 whitespace-nowrap text-sm font-semibold text-gray-900 border-gray-300 border-r">{employee.hoursWorked}</td>
                                        </tr>
                                        {/each}
                                        {/if}
                                </tbody>
                            </table>
                        </div>

                        <!-- Pagination -->
                        <!-- {#if filteredEmployees.length > itemsPerPage} -->
                            <div class="flex flex-col sm:flex-row items-center justify-between pt-4 border-t border-gray-200">
                                <div class="text-base font-semibold text-gray-700 mb-2 sm:mb-0">
                                    Showing <span class="font-medium">{(currentPage - 1) * itemsPerPage + 1}</span> to 
                                    <span class="font-medium">{Math.min(currentPage * itemsPerPage, filteredEmployees.length)}</span> of 
                                    <span class="font-medium">{filteredEmployees.length}</span> results
                                </div>
                                
                                <div class="flex space-x-1">
                                    <button
                                        on:click={() => currentPage = Math.max(1, currentPage - 1)}
                                        disabled={currentPage === 1}
                                        class="px-3 py-1 rounded-md text-base font-semibold text-gray-500 disabled:opacity-50"
                                    >
                                        Previous
                                    </button>
                                    
                                    {#each Array(totalPages).fill(0) as _, i}
                                        {#if i + 1 === currentPage || 
                                           i + 1 === currentPage - 1 || 
                                           i + 1 === currentPage + 1 ||
                                           i === 0 ||
                                           i === totalPages - 1}
                                            <button
                                                on:click={() => currentPage = i + 1}
                                                class={`px-3 py-1 border text-sm font-medium ${
                                                    currentPage === i + 1
                                                        ? 'bg-gray-200 border-[#02066F] cursor-pointer focus:outline-none focus:ring-1 focus:ring-[#02066F] '
                                                        : 'bg-white border-gray-300 text-gray-700 hover:bg-gray-50'
                                                } rounded-sm`}
                                            >
                                                {i + 1}
                                            </button>
                                        {:else if (i + 1 === currentPage - 2 || i + 1 === currentPage + 2)}
                                            <span class="px-3 py-1 text-gray-700">...</span>
                                        {/if}
                                    {/each}
                                    
                                    <button
                                        on:click={() => currentPage = Math.min(totalPages, currentPage + 1)}
                                        disabled={currentPage === totalPages}
                                        class="px-3 py-1 rounded-md text-base font-semibold text-gray-500 disabled:opacity-50"
                                    >
                                        Next
                                    </button>
                                </div>
                            </div>
                        <!-- {/if} -->
                    <!-- {/if} -->
                </div>
            </div>
        </div>
    </main>
</div>
</div>