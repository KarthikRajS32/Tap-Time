<script>
    import { onMount } from 'svelte';

    const apiUrlBase = 'https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/report/dateRangeReportGet';

    // State
    let startDateHeader = '';
    let endDateHeader = '';
    let reportName = '';
    let reportTypeHeading = '';
    // @ts-ignore
    let employees = [];
    let isLoading = true;
    let showHomeModal = false;
    let showLogoutModal = false;
    let searchTerm = '';
    let sortConfig = { key: 'name', direction: 'asc' };

    // Computed properties
   // @ts-ignore
     $: filteredEmployees = employees
        .filter(emp => 
            emp.name.toLowerCase().includes(searchTerm.toLowerCase()) || 
            emp.pin.toLowerCase().includes(searchTerm.toLowerCase())
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
                .map(([pin, empData]) => ({
                    pin,
                    name: empData.name,
                    hoursWorked: empData.totalHoursWorked
                }));
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

    // Date calculation functions remain the same as previous example
    // ... (getLastWeekDateRange, getLastTwoWeeksDateRange, etc.)

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

        for (const [pin, details] of Object.entries(employeeTimes)) {
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

</script>

<div class="bg-gray-100">
<div class="pt-16 md:pt-18 sm:pt-2">
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
        <h1 class="text-2xl font-bold text-center mb-8 ">{reportTypeHeading}</h1>

        <!-- Date Range Display -->
        <div class="flex flex-col max-w-5xl mx-auto md:flex-row justify-between mb-6 p-4 rounded-lg ">
            <div class="mb-2 md:mb-0">
                <span class="text-lg font-semibold">Start Date: </span>
                <span>{startDateHeader}</span>
            </div>
            <div>
                <span class="text-lg font-semibold">End Date: </span>
                <span>{endDateHeader}</span>
            </div>
        </div>

        <!-- Search and Controls -->
        <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-sm overflow-hidden mb-8 border-1 border-gray-300">
            <div class="p-4 sm:p-6 overflow-x-auto">
           
            <!-- <div class="flex space-x-2">
                <button 
                    on:click={() => showHomeModal = true}
                    class="px-4 py-2 bg-white border border-blue-900 text-blue-900 rounded-lg hover:bg-blue-50 transition"
                >
                    Home
                </button>
                <button 
                    on:click={() => showLogoutModal = true}
                    class="px-4 py-2 bg-white border border-blue-900 text-blue-900 rounded-lg hover:bg-blue-50 transition"
                >
                    Logout
                </button>
            </div> -->
        

        <!-- Employee Table -->
        <div class="overflow-hidden">
            {#if filteredEmployees.length === 0}
                <div class="text-center p-8 text-gray-700">
                    {searchTerm ? 'No matching employees found' : 'No records available'}
                </div>
            {:else}
             <!-- Search -->
             <div class="pb-4 flex flex-col sm:flex-row sm:items-center sm:justify-end gap-2">
                <label for="search" class="text-sm font-medium text-gray-700">Search:</label>
                <input
                  id="search"
                  type="text"
                  bind:value={searchTerm}
                  class="w-full sm:w-64 h-9 pl-2 border border-gray-400 rounded-md focus:outline-none focus:ring-1 focus:ring-black"
                />
              </div>
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-blue-900 text-white">
                            <tr>
                                <th 
                                    class="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider cursor-pointer"
                                    on:click={() => requestSort('name')}
                                >
                                    Name
                                    {#if sortConfig.key === 'name'}
                                        <span class="ml-1">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                    {/if}
                                </th>
                                <th 
                                    class="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider cursor-pointer"
                                    on:click={() => requestSort('pin')}
                                >
                                    Pin
                                    {#if sortConfig.key === 'pin'}
                                        <span class="ml-1">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                    {/if}
                                </th>
                                <th 
                                    class="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider cursor-pointer"
                                    on:click={() => requestSort('hoursWorked')}
                                >
                                   Total Worked Hours (HH:MM)
                                    {#if sortConfig.key === 'hoursWorked'}
                                        <span class="ml-1">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                    {/if}
                                </th>
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200">
                            {#each filteredEmployees as employee}
                                <tr class="hover:bg-gray-50">
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{employee.name}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{employee.pin}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{employee.hoursWorked}</td>
                                </tr>
                            {/each}
                        </tbody>
                    </table>
                </div>
            {/if}
        </div>
        </div>
    </div>
    </main>
</div>
</div>