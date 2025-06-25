<script lang="ts">
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';
    // Use a string for the report type heading
    let reportTypeHeading: string = '';

    // State variables
    let selectedDate: string = '';
    let isLoading: boolean = false;
    let reportData: any[] = [];
    let filteredData: any[] = [];
    let searchQuery: string = '';
    let currentPage: number = 1;
    const itemsPerPage = 10;
    let sortColumn: string = '';
    let sortDirection: 'asc' | 'desc' = 'asc';
    
    const apiUrlBase = 'https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/dailyreport/getdatebasedata';
    let cid: string | null = localStorage.getItem('companyID');
    let sidebarOpen: boolean = false;
    let reportName: string = localStorage.getItem('reportType') || 'Salaried';

   

    // Close sidebar when clicking outside
    const handleClickOutside = (event: MouseEvent) => {
        const sidebar = document.getElementById('sidebar');
        const toggler = document.querySelector('.navbar-toggler');
        
        if (sidebar && toggler && !sidebar.contains(event.target as Node) && !toggler.contains(event.target as Node)) {
            sidebarOpen = false;
        }
    };

    // Format date time to AM/PM
    const convertToAmPm = (dateString: string) => {
        const date = new Date(dateString);
        let hours = date.getHours();
        const minutes = date.getMinutes();
        const ampm = hours >= 12 ? 'PM' : 'AM';
        hours = hours % 12;
        hours = hours ? hours : 12;
        const minutesStr = minutes < 10 ? '0' + minutes : minutes;
        return `${hours}:${minutesStr} ${ampm}`;
    };

    // Calculate time worked
    const calculateTimeWorked = (checkIn: string, checkOut: string) => {
        const checkInTime = new Date(checkIn);
        const checkOutTime = new Date(checkOut);
        const diffMs = checkOutTime.getTime() - checkInTime.getTime();
        
        const hours = Math.floor(diffMs / (1000 * 60 * 60));
        const minutes = Math.floor((diffMs % (1000 * 60 * 60)) / (1000 * 60));
        
        return `${hours}:${minutes.toString().padStart(2, '0')}`;
    };

    // View date-wise report
    const viewDatewiseReport = async (dateValue: string) => {
        if (!dateValue) {
            reportData = [];
            filteredData = [];
            return;
        }

        isLoading = true;
        
        try {
            const apiUrl = `${apiUrlBase}/${cid}/${dateValue}`;
            const response = await fetch(apiUrl);
            
            if (!response.ok) {
                throw new Error(`Error: ${response.status}`);
            }
            
            const data = await response.json();
            reportData = data.map((item: any) => ({
                ...item,
                formattedCheckIn: item.CheckInTime ? convertToAmPm(item.CheckInTime) : '--',
                formattedCheckOut: item.CheckOutTime ? convertToAmPm(item.CheckOutTime) : '--',
                timeWorked: item.CheckInTime && item.CheckOutTime ? calculateTimeWorked(item.CheckInTime, item.CheckOutTime) : '--'
            }));
            
            filteredData = [...reportData];
            currentPage = 1;
        } catch (error) {
            console.error('Error:', error);
            reportData = [];
            filteredData = [];
        } finally {
            isLoading = false;
        }
    };

    // Sort data
    const sortData = (column: string) => {
        if (sortColumn === column) {
            sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
        } else {
            sortColumn = column;
            sortDirection = 'asc';
        }

        filteredData.sort((a, b) => {
            const valA = a[column];
            const valB = b[column];
            
            if (valA < valB) return sortDirection === 'asc' ? -1 : 1;
            if (valA > valB) return sortDirection === 'asc' ? 1 : -1;
            return 0;
        });
    };

    // Filter data based on search
    const filterData = () => {
        if (!searchQuery) {
            filteredData = [...reportData];
            return;
        }
        
        const query = searchQuery.toLowerCase();
        filteredData = reportData.filter(item => 
            item.Name.toLowerCase().includes(query) || 
            item.Pin.toLowerCase().includes(query)
        );
        currentPage = 1;
    };

    // Pagination
    const paginatedData = () => {
        const start = (currentPage - 1) * itemsPerPage;
        return filteredData.slice(start, start + itemsPerPage);
    };

    const totalPages = Math.ceil(filteredData.length / itemsPerPage);


    // Initialize with today's date
    onMount(() => {
        document.addEventListener('click', handleClickOutside);
        
        const today = new Date();
        const formattedDate = today.toISOString().split('T')[0];
        // selectedDate = formattedDate;
        viewDatewiseReport(formattedDate);
        
        return () => {
            document.removeEventListener('click', handleClickOutside);
        };
    });

    onMount(() => {
        const selectedValue = localStorage.getItem('reportType');
        reportName = `${selectedValue} Report`;
        reportTypeHeading = `${selectedValue} Report`;
       
    });
</script>

<div class="bg-gray-100">

    <!-- Main Content -->
    <div class="pt-16 md:pt-18 sm:pt-2">
        
         <!-- Navigation -->
         <nav class="bg-white shadow">
            <div class="max-w-8xl mx-auto px-4 sm:px-6 lg:px-8">
              <div class="flex flex-col md:flex-row justify-between items-center h-auto md:h-16 py-4 md:py-0 justify-end">
                
                <!-- Menu Links - visible on all devices -->
                <div class="flex flex-col md:flex-row space-y-2 md:space-y-0 md:space-x-4 text-center w-auto md:w-auto">
                  <a href="/reportsummary" class="px-4 py-2 text-[#02066F]  font-semibold rounded-full">Today's Report</a>
                  <a href="/daywisereport" class="px-4 py-2 bg-[#02066F] text-white font-semibold rounded-full">Day Wise Report</a>
                  <a href="/salariedreport" class="px-4 py-2 text-[#02066F] font-semibold rounded-full">{reportTypeHeading}</a>
                </div>
          
              </div>
            </div>
          </nav>
        <!-- Date Picker -->
        <div class="flex justify-center my-8">
            <input 
                type="date" 
                bind:value={selectedDate}
                on:change={() => viewDatewiseReport(selectedDate)}
                class="border bg-white border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 shadow"
            />
        </div>

        <!-- Search and Table -->
        {#if selectedDate}
            <div class="border-gray-300 rounded-xl overflow-hidden mb-auto">
               

                <!-- Report Table -->
                <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-md overflow-hidden mb-8 border border-gray-300">
                    <div class="p-4 sm:p-6 overflow-x-auto">
                
                <!-- Search -->
                <div class="pb-4 flex flex-col sm:flex-row sm:items-center sm:justify-end gap-2">
                    <label for="search" class="text-sm font-medium text-gray-700">Search:</label>
                    <input
                      id="search"
                      type="text"
                      bind:value={searchQuery}
                      on:input={filterData}
                      class="w-full sm:w-64 h-9 pl-2 border border-gray-400 rounded-md focus:outline-none focus:ring-1 focus:ring-black"
                    />
                  </div>
                    <table class="min-w-full border border-gray-300 text-sm">
                        <thead class="bg-[#02066F] text-white">
                            <tr>
                                <th 
                                    class="px-4 sm:px-6 py-3 text-left font-semibold border-r tracking-wider cursor-pointer"
                                    on:click={() => sortData('Name')}
                                >
                                    <div class="flex items-center justify-center">
                                        Employee Name
                                        {#if sortColumn === 'Name'}
                                            <span class="ml-1">
                                                {sortDirection === 'asc' ? '↑' : '↓'}
                                            </span>
                                        {/if}
                                    </div>
                                </th>
                                <th 
                                    class="px-4 sm:px-6 py-3 text-left font-semibold border-r tracking-wider cursor-pointer"
                                    on:click={() => sortData('Pin')}
                                >
                                    <div class="flex items-center justify-center">
                                        Employee ID
                                        {#if sortColumn === 'Pin'}
                                            <span class="ml-1">
                                                {sortDirection === 'asc' ? '↑' : '↓'}
                                            </span>
                                        {/if}
                                    </div>
                                </th>
                                <th class="px-4 sm:px-6 py-3 text-left font-semibold border-r tracking-wider">
                                    Check-in Time
                                </th>
                                <th class="px-4 sm:px-6 py-3 text-left font-semibold border-r tracking-wider">
                                    Check-out Time
                                </th>
                                <th class="px-4 sm:px-6 py-3 text-left font-semibold border-r tracking-wider">
                                    Time Worked Hours (HH:MM)
                                </th>
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200">
                            {#if filteredData.length === 0}
                                <tr>
                                    <td colspan="5" class="px-4 py-4 text-center text-gray-500">
                                        {reportData.length === 0 ? 'No records found for selected date' : 'No matching records found'}
                                    </td>
                                </tr>
                            {:else}
                                {#each paginatedData() as item (item.Pin)}
                                    <tr class="hover:bg-gray-50">
                                        <td class="px-4 py-4 text-center text-sm text-gray-700">
                                            {item.Name.split(" ")[0]}
                                        </td>
                                        <td class="px-4 py-4 text-center text-sm text-gray-700">
                                            {item.Pin}
                                        </td>
                                        <td class="px-4 py-4 text-center text-sm text-gray-700">
                                            {item.formattedCheckIn}
                                        </td>
                                        <td class="px-4 py-4 text-center text-sm text-gray-700">
                                            {item.formattedCheckOut}
                                        </td>
                                        <td class="px-4 py-4 text-center text-sm text-gray-700">
                                            {item.timeWorked}
                                        </td>
                                    </tr>
                                {/each}
                            {/if}
                        </tbody>
                    </table>
                    </div>
                </div>

                <!-- Pagination -->
                {#if filteredData.length > itemsPerPage}
                    <div class="px-4 py-3 flex items-center justify-between border-t border-gray-200">
                        <div class="flex-1 flex justify-between sm:hidden">
                            <button
                                on:click={() => currentPage = Math.max(1, currentPage - 1)}
                                disabled={currentPage === 1}
                                class="relative inline-flex items-center px-4 py-2 border border-gray-300 text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 disabled:opacity-50"
                            >
                                Previous
                            </button>
                            <button
                                on:click={() => currentPage = Math.min(totalPages, currentPage + 1)}
                                disabled={currentPage === totalPages}
                                class="ml-3 relative inline-flex items-center px-4 py-2 border border-gray-300 text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 disabled:opacity-50"
                            >
                                Next
                            </button>
                        </div>
                        <div class="hidden sm:flex-1 sm:flex sm:items-center sm:justify-between">
                            <div>
                                <p class="text-sm text-gray-700">
                                    Showing <span class="font-medium">{(currentPage - 1) * itemsPerPage + 1}</span> to 
                                    <span class="font-medium">{
                                        Math.min(currentPage * itemsPerPage, filteredData.length)
                                    }</span> of 
                                    <span class="font-medium">{filteredData.length}</span> results
                                </p>
                            </div>
                            <div>
                                <nav class="relative z-0 inline-flex rounded-md shadow-sm -space-x-px" aria-label="Pagination">
                                    <button
                                        on:click={() => currentPage = Math.max(1, currentPage - 1)}
                                        disabled={currentPage === 1}
                                        class="relative inline-flex items-center px-2 py-2 rounded-l-md border border-gray-300 bg-white text-sm font-medium text-gray-500 hover:bg-gray-50 disabled:opacity-50"
                                    >
                                        <span class="sr-only">Previous</span>
                                        &larr;
                                    </button>
                                    {#each Array(totalPages) as _, i}
                                        <button
                                            on:click={() => currentPage = i + 1}
                                            class={`relative inline-flex items-center px-4 py-2 border text-sm font-medium ${
                                                currentPage === i + 1 
                                                    ? 'z-10 bg-blue-900 border-blue-900 text-white' 
                                                    : 'bg-white border-gray-300 text-gray-500 hover:bg-gray-50'
                                            }`}
                                        >
                                            {i + 1}
                                        </button>
                                    {/each}
                                    <button
                                        on:click={() => currentPage = Math.min(totalPages, currentPage + 1)}
                                        disabled={currentPage === totalPages}
                                        class="relative inline-flex items-center px-2 py-2 rounded-r-md border border-gray-300 bg-white text-sm font-medium text-gray-500 hover:bg-gray-50 disabled:opacity-50"
                                    >
                                        <span class="sr-only">Next</span>
                                        &rarr;
                                    </button>
                                </nav>
                            </div>
                        </div>
                    </div>
                {/if}
            </div>
            {:else}
            <div class="border-gray-300 rounded-xl overflow-hidden mb-auto">
                <p class="p-14 text-center text-gray-700 ">Please select a date to show report.</p>
            </div>
        {/if}
    </div>
</div>