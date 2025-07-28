<script lang="ts">
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';
    import jsPDF from 'jspdf';
    // Import autoTable like this:
    import autoTable from 'jspdf-autotable';

    let reportTypeHeading: string = '';
    let selectedDate: string = '';
    let loading = false;
    let reportData: any[] = [];
    let filteredData: any[] = [];
    let searchQuery = '';
    let currentPage = 1;
    let itemsPerPage = 10;
    let sortColumn = '';
    let sortDirection: 'asc' | 'desc' = 'asc';
    
    // Device dropdown state
    let devices: any[] = [];
    let selectedDevice: any = null;

    const adminType = localStorage.getItem('adminType');
    const deviceID = localStorage.getItem('DeviceID');
    
    // Report frequency state - Load immediately, not async
    let availableFrequencies: string[] = loadFrequenciesSync();
    let selectedFrequency: string = availableFrequencies[0] || '';
    
    // Synchronous function to load frequencies immediately
    function loadFrequenciesSync(): string[] {
        const savedFrequencies = localStorage.getItem('reportType');
        if (savedFrequencies) {
            return savedFrequencies.split(',').filter(f => f.trim() !== '');
        }
        return [];
    }

    const apiUrlBase = 'https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/dailyreport/getdatebasedata';
    const deviceApiUrl = 'https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/device';
    const cid: string | null = localStorage.getItem('companyID');
    let reportName: string = localStorage.getItem('reportType') || 'Salaried';

    const convertToAmPm = (dateString: string) => {
        const date = new Date(dateString);
        let hours = date.getHours();
        const minutes = date.getMinutes();
        const ampm = hours >= 12 ? 'PM' : 'AM';
        hours = hours % 12 || 12;
        const minutesStr = minutes < 10 ? '0' + minutes : minutes;
        return `${hours}:${minutesStr} ${ampm}`;
    };

    const calculateTimeWorked = (checkIn: string, checkOut: string) => {
        const inTime = new Date(checkIn);
        const outTime = new Date(checkOut);
        const diff = outTime.getTime() - inTime.getTime();
        const hours = Math.floor(diff / (1000 * 60 * 60));
        const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
        return `${hours}:${minutes.toString().padStart(2, '0')}`;
    };

    // Fetch device data
    const fetchDevices = async () => {
        try {
            const response = await fetch(`${deviceApiUrl}/getAll/${cid}`);
            if (!response.ok) throw new Error(`Error: ${response.status}`);
            
            const data = await response.json();
            // Filter out devices with "Not Registered" names
            const allDevices = Array.isArray(data) ? data : [data];
            devices = allDevices.filter(device => 
                device.DeviceName && 
                device.DeviceName !== "Not Registered" && 
                device.DeviceName.trim() !== ""
            );
            
            // Set first valid device as default selection
            if (devices.length > 0) {
                selectedDevice = devices[0];
                console.log('Default selected device:', selectedDevice);
            }
        } catch (error) {
            console.error('Error fetching devices:', error);
        }
    };

    // Handle device selection
    const handleDeviceSelection = (device: any) => {
        selectedDevice = device;
        console.log('Selected device for reports:', device);
        // Refresh report with new device filter
        if (selectedDate) {
            viewDatewiseReport(selectedDate);
        }
    };

    const viewDatewiseReport = async (dateValue: string) => {
    if (!dateValue || !cid) {
        reportData = [];
        filteredData = [];
        return;
    }

    loading = true;
    try {
        const res = await fetch(`${apiUrlBase}/${cid}/${dateValue}`);
        if (!res.ok) throw new Error(`Error: ${res.status}`);
        const data = await res.json();

        // ✅ Ensure you safely get an array (based on your backend)
        const records = Array.isArray(data) 
            ? data 
            : Array.isArray(data.body) 
                ? data.body 
                : [];

        // ✅ Guard against undefined records
        if (!Array.isArray(records)) throw new Error('Invalid API data');

        let processedData = records.map((item: any) => ({
            ...item,
            formattedCheckIn: item.CheckInTime ? convertToAmPm(item.CheckInTime) : '--',
            formattedCheckOut: item.CheckOutTime ? convertToAmPm(item.CheckOutTime) : '--',
            timeWorked: item.CheckInTime && item.CheckOutTime
                ? calculateTimeWorked(item.CheckInTime, item.CheckOutTime)
                : '--'
        }));
        if(adminType != 'Owner')
        {
          processedData = processedData.filter(item => item.DeviceID === deviceID);
          console.log(`Filtered today's report by DeviceID ${deviceID}:`, processedData.length, 'records');
        }

        // Filter by selected device if available
        else if (selectedDevice && selectedDevice.DeviceID) {
            processedData = processedData.filter(item => item.DeviceID === selectedDevice.DeviceID);
            console.log(`Filtered report by DeviceID ${selectedDevice.DeviceID}:`, processedData.length, 'records');
        }

        reportData = processedData;
        filteredData = [...reportData];
        currentPage = 1;
        sortColumn = '';
        sortDirection = 'asc';
    } catch (err) {
        console.error('Error fetching report:', err);
        reportData = [];
        filteredData = [];
    } finally {
        loading = false;
    }
};

    const filterData = () => {
        if (!searchQuery) {
            filteredData = [...reportData];
        } else {
            const query = searchQuery.toLowerCase();
            filteredData = reportData.filter(item =>
                (item.Name?.toLowerCase().includes(query) || item.Pin?.toLowerCase().includes(query))
            );
        }
        currentPage = 1;
    };

    const paginatedData = () => {
        const start = (currentPage - 1) * itemsPerPage;
        return filteredData.slice(start, start + itemsPerPage);
    };

    const totalPages = () => Math.ceil(filteredData.length / itemsPerPage);

    const downloadPDF = () => {
        const doc = new jsPDF();
        
        doc.text(`Day-Wise Report - ${selectedDate}`, 14, 10);

        const tableColumn = ["Employee Name", "Employee ID", "Check-in Time", "Check-out Time", "Time Worked Hours(HH:MM)"];
        const tableRows = reportData.map(item => [
            item.Name?.split(" ")[0] || '',
            item.Pin || "--",
            item.formattedCheckIn || "--",
            item.formattedCheckOut || "--",
            item.timeWorked || "--"
        ]);
        
        // Use autoTable directly (no need for @ts-ignore)
        autoTable(doc, {
            head: [tableColumn],
            body: tableRows,
            startY: 20,
            headStyles: {
            fillColor: [2, 6, 111], // Dark blue header
            textColor: 255, // White text
            fontSize: 10,
            fontStyle: 'bold',
            },
            styles: { 
                fontSize: 10,
                // cellPadding: 3,
             },
            theme: 'grid'
        });

        doc.save(`DayWise_Report_${new Date().toISOString().split("T")[0]}.pdf`);
    };

    const downloadCSV = () => {
        if (filteredData.length === 0) {
            alert('No data to download');
            return;
        }

        const headers = ['Employee Name', 'Employee ID', 'Check-in Time', 'Check-out Time', 'Time Worked'];
        const csvRows = filteredData.map(item => [
            `"${item.Name?.split(" ")[0] ?? ''}"`,
            item.Pin ?? '',
            item.formattedCheckIn,
            item.formattedCheckOut,
            item.timeWorked
        ]);

        const csvContent = [headers.join(','), ...csvRows.map(row => row.join(','))].join('\n');
        const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
        const url = URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = url;
        link.download = `day_wise_report_${selectedDate || 'report'}.csv`;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    };

    onMount(async () => {
        const today = new Date();
        const formattedDate = today.toISOString().split('T')[0];
        selectedDate = formattedDate;
        
        await fetchDevices();
        viewDatewiseReport(formattedDate);

        const selectedValue = localStorage.getItem('reportType') || 'Salaried';
        reportName = `${selectedValue} Report`;
        reportTypeHeading = `${selectedValue} Report`;
    });


    let sortConfig = {
        key: '',
        direction: 'asc' as 'asc' | 'desc'
    };

    const requestSort = (key: string) => {
        // 1. Determine new direction
        let direction: 'asc' | 'desc' = 'asc';
        if (sortConfig.key === key) {
            direction = sortConfig.direction === 'asc' ? 'desc' : 'asc';
        }

        // 2. Update sort configuration
        sortConfig = { key, direction };

        // 3. Apply the sort
        filteredData = [...filteredData].sort((a, b) => {
            // Handle missing values
            const valA = a[key] ?? '';
            const valB = b[key] ?? '';

            // Special handling for date fields
            if (key.toLowerCase().includes('time')) {
                const dateA = valA ? new Date(valA).getTime() : 0;
                const dateB = valB ? new Date(valB).getTime() : 0;
                return direction === 'asc' ? dateA - dateB : dateB - dateA;
            }
            

            // Numeric comparison
            if (!isNaN(Number(valA))) {
                return direction === 'asc' 
                    ? Number(valA) - Number(valB) 
                    : Number(valB) - Number(valA);
            }

            // Default string comparison
            return direction === 'asc'
                ? String(valA).localeCompare(String(valB))
                : String(valB).localeCompare(String(valA));
        });
    };

</script>



<div class="bg-gray-100">

    {#if loading}
        <div class="fixed inset-0 flex items-center justify-center z-50"    
        style="background: rgba(0, 0, 0, 0.5)">
        <div class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"></div>
        </div>
    {/if}

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
                        
                        <!-- Show separate frequency tabs -->
                        {#if availableFrequencies.length > 1}
                          {#each availableFrequencies as frequency}
                            <a href="/salariedreport" class="px-4 py-2 text-[#02066F] font-semibold rounded-full">
                              {frequency} Report
                            </a>
                          {/each}
                        {:else if availableFrequencies.length === 1}
                          <a href="/salariedreport" class="px-4 py-2 text-[#02066F] font-semibold rounded-full">
                            {availableFrequencies[0]} Report
                          </a>
                        {/if}
                    </div>
                </div>
            </div>
        </nav>

        <!-- Device Dropdown Section -->
        <div class="max-w-5xl mx-auto mb-6 px-4">
            <div class="flex justify-center">
                <div class="relative inline-block text-left">
                    {#if adminType == 'Owner'}
                    <div>
                        <button
                            type="button"
                            class="inline-flex w-full justify-center gap-x-1.5 rounded-md bg-white px-4 py-2 text-sm font-semibold text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 hover:bg-gray-50"
                            id="device-menu-button"
                            aria-expanded="true"
                            aria-haspopup="true"
                            on:click={() => {
                                const dropdown = document.getElementById('device-dropdown');
                                dropdown.classList.toggle('hidden');
                            }}
                        >
                            {selectedDevice ? selectedDevice.DeviceName : 'Select Device Name'}
                            <svg class="-mr-1 h-5 w-5 text-gray-400" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                                <path fill-rule="evenodd" d="M5.23 7.21a.75.75 0 011.06.02L10 11.168l3.71-3.938a.75.75 0 111.08 1.04l-4.25 4.5a.75.75 0 01-1.08 0l-4.25-4.5a.75.75 0 01.02-1.06z" clip-rule="evenodd" />
                            </svg>
                        </button>
                    </div>
                    {/if}
                    <div 
                        id="device-dropdown"
                        class="absolute right-0 z-10 mt-2 w-56 origin-top-right rounded-md bg-white shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none hidden"
                        role="menu" 
                        aria-orientation="vertical" 
                        aria-labelledby="device-menu-button" 
                        tabindex="-1"
                    >
                        <div class="py-1" role="none">
                            {#each devices as device}
                                <button
                                    type="button"
                                    class="text-gray-700 block w-full px-4 py-2 text-left text-sm hover:bg-gray-100 hover:text-gray-900"
                                    role="menuitem"
                                    tabindex="-1"
                                    on:click={() => {
                                        handleDeviceSelection(device);
                                        document.getElementById('device-dropdown').classList.add('hidden');
                                    }}
                                >
                                    {device.DeviceName}
                                </button>
                            {:else}
                                <div class="text-gray-500 block px-4 py-2 text-sm">No devices available</div>
                            {/each}
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Date Picker and Download Buttons -->
        <div class="flex flex-col md:flex-row justify-center items-center my-8 gap-4 px-4">
            <input 
                type="date" 
                bind:value={selectedDate}
                on:change={() => viewDatewiseReport(selectedDate)}
                class="border bg-white border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 shadow"
            />
            
            
        </div>

        <!-- Search and Table -->
        {#if selectedDate}
            <div class="border-gray-300 rounded-xl overflow-hidden mb-auto px-4">
               
                <!-- Report Table -->
                <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-md overflow-hidden mb-8 border border-gray-300">
                    <div class="p-4 sm:p-6 overflow-x-auto">

                        <!-- Download Buttons -->
                        <!-- {#if filteredData.length > 0} -->
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
                        <!-- {#if filteredData.length > 0} -->
                        <div class="flex flex-col sm:flex-row justify-between items-center gap-4 mb-4">
                            <div class="flex items-center gap-2">
                                <label for="entries" class="text-base font-semibold text-gray-700">Show</label>
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
                                    bind:value={searchQuery}
                                    on:input={filterData}
                                    placeholder=""
                                    class="w-full sm:w-64 px-2 py-1 border border-gray-500 rounded-md focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                                />
                            </div>
                        </div>
                        <!-- {/if} -->
                        
                        <table class="min-w-full border border-gray-300 text-sm overflow-x-auto">
                            <thead class="bg-[#02066F] text-white">
                                <tr>
                                    <th 
                                    class="px-4 sm:px-6 py-3 text-center font-semibold border-r tracking-wider cursor-pointer"
                                    on:click={() => requestSort('Name')}
                                    >
                                        <div class="flex items-center justify-center">
                                            Employee Name
                                            {#if sortConfig.key === 'Name'}
                                                <span class="ml-6 text-lg">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                            {:else}
                                                <span class="ml-6 text-lg">↑↓</span>
                                            {/if}
                                        </div>
                                    </th>
                                    <th 
                                    class="px-4 sm:px-6 py-3 text-center font-semibold border-r tracking-wider cursor-pointer"
                                    on:click={() => requestSort('Pin')}
                                    >
                                        <div class="flex items-center justify-center">
                                            Employee ID
                                            {#if sortConfig.key === 'Pin'}
                                                <span class="ml-6 text-lg">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                            {:else}
                                                <span class="ml-6 text-lg">↑↓</span>
                                            {/if}
                                        </div>
                                    </th>
                                    <th 
                                        class="px-4 sm:px-6 py-3 text-center font-semibold border-r tracking-wider cursor-pointer"
                                        on:click={() => requestSort('CheckInTime')}
                                    >
                                        <div class="flex items-center justify-center">
                                            Check-in Time
                                            {#if sortConfig.key === 'CheckInTime'}
                                                <span class="ml-6 text-lg">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                            {:else}
                                                <span class="ml-6 text-lg">↑↓</span>
                                            {/if}
                                        </div>
                                    </th>
                                    <th 
                                        class="px-4 sm:px-6 py-3 text-center font-semibold border-r tracking-wider cursor-pointer"
                                        on:click={() => requestSort('CheckOutTime')}
                                    >
                                        
                                        <div class="flex items-center justify-center">
                                            Check-out Time
                                            {#if sortConfig.key === 'CheckOutTime'}
                                                <span class="ml-6 text-lg">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                            {:else}
                                                <span class="ml-6 text-lg">↑↓</span>
                                            {/if}
                                        </div>
                                    </th>
                                    <th 
                                        class="px-4 sm:px-6 py-3 text-center font-semibold border-r tracking-wider cursor-pointer"
                                        on:click={() => requestSort('timeWorked')}
                                    >
                                        
                                        <div class="flex items-center justify-center">
                                            Time Worked Hours (HH:MM)
                                            {#if sortConfig.key === 'timeWorked'}
                                                <span class="ml-6 text-lg">{sortConfig.direction === 'asc' ? '↑' : '↓'}</span>
                                            {:else}
                                                <span class="ml-6 text-lg">↑↓</span>
                                            {/if}
                                        </div>
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
                                {#each paginatedData() as item, index (item.Pin + '-' + index)}
                                        <tr class="hover:bg-gray-50">
                                            <td class="px-4 py-3 whitespace-nowrap text-center font-semibold text-gray-900 text-sm border-gray-300 border-r">
                                                          
                                                      
                                                {item.Name}
                                            </td>
                                            <td class="px-4 py-3 whitespace-nowrap text-center font-semibold text-gray-900 text-sm border-gray-300 border-r">
                                                {item.Pin}
                                            </td>
                                            <td class="px-4 py-3 whitespace-nowrap text-center font-semibold text-gray-900 text-sm border-gray-300 border-r">
                                                {item.formattedCheckIn}
                                            </td>
                                            <td class="px-4 py-3 whitespace-nowrap text-center font-semibold text-gray-900 text-sm border-gray-300 border-r">
                                                {item.formattedCheckOut}
                                            </td>
                                            <td class="px-4 py-3 whitespace-nowrap text-center font-semibold text-gray-900 text-sm border-gray-300 border-r">
                                                {item.timeWorked}
                                            </td>
                                        </tr>
                                    {/each}
                                {/if}
                            </tbody>
                        </table>
                        
                        <!-- Pagination -->
                        <!-- {#if filteredData.length > 0} -->
                            <div class="flex flex-col sm:flex-row items-center justify-between pt-4 border-t border-gray-200">
                                <div class="text-base font-semibold text-gray-700 mb-2 sm:mb-0">
                                    Showing <span class="font-medium">{(currentPage - 1) * itemsPerPage + 1}</span> to 
                                    <span class="font-medium">{Math.min(currentPage * itemsPerPage, filteredData.length)}</span> of 
                                    <span class="font-medium">{filteredData.length}</span> results
                                </div>
                                
                                <div class="flex space-x-1">
                                    <button
                                        on:click={() => currentPage = Math.max(1, currentPage - 1)}
                                        disabled={currentPage === 1}
                                        class="px-3 py-1 rounded-md text-base font-semibold text-gray-500 disabled:opacity-50"
                                    >
                                        Previous
                                    </button>
                                    
                                    {#each Array(totalPages()).fill(0) as _, i}
                                        {#if i + 1 === currentPage || 
                                           i + 1 === currentPage - 1 || 
                                           i + 1 === currentPage + 1 ||
                                           i === 0 ||
                                           i === totalPages() - 1}
                                            <button
                                                on:click={() => currentPage = i + 1}
                                                class={`px-3 py-1 border text-sm font-medium ${
                                                    currentPage === i + 1
                                                        ? 'bg-gray-200 border-[#02066F]'
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
                                        on:click={() => currentPage = Math.min(totalPages(), currentPage + 1)}
                                        disabled={currentPage === totalPages()}
                                        class="px-3 py-1 rounded-md text-base font-semibold text-gray-500 disabled:opacity-50"
                                    >
                                        Next
                                    </button>
                                </div>
                            </div>
                        <!-- {/if} -->
                    </div>
                </div>
            </div>
            {:else}
            <div class="border-gray-300 rounded-xl overflow-hidden mb-auto">
                <p class="p-14 text-center text-gray-700 ">Please select a date to show report.</p>
            </div>
        {/if}
    </div>
</div>