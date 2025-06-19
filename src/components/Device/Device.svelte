<script>
    import { onMount } from 'svelte';
    import { v4 as uuidv4 } from 'uuid';
    // import * as bootstrap from 'bootstrap';
    // import Modal from './Modal.svelte';

    // State variables
    let showTable = false;
    let showNoDeviceMessage = false;
    let isLoading = false;
    /** @type {{
        TimeZone: string;
        DeviceID: string;
        CID: string;
        DeviceName: string;
        AccessKey: string;
        AccessKeyGeneratedTime: string;
        LastModifiedBy: string;
    }[]} */
    let devices = [];
    let showHomeModal = false;
    let showLogoutModal = false;
    let showDeleteModal = false;
    let deviceToDelete = '';
    let sidebarOpen = false;

    localStorage.setItem('companyID', '1234567890'); // Example company ID for testing

    // API configuration
    const apiUrlBase = "https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/device";

    // Initialize on component mount
    onMount(() => {
        viewDevices();
    });


    function confirmHomeNavigation() {
        window.open('/', 'noopener, noreferrer');
    }


    // Device management functions
    // @ts-ignore
    function maskString(input, visibleChars) {
        visibleChars = Math.min(visibleChars, input.length);
        const starsCount = input.length - visibleChars;
        const stars = '*'.repeat(starsCount);
        const visiblePart = input.slice(-visibleChars);
        return stars + visiblePart;
    }

    // @ts-ignore
    function copyAccessKey(accessKeyValue) {
        navigator.clipboard.writeText(accessKeyValue).then(() => {
            alert('Copied to clipboard!');
        }).catch(err => {
            console.error('Failed to copy:', err);
        });
    }

    

   async function viewDevices() {
    isLoading = true;
    const companyId = localStorage.getItem('companyID');

    if (!companyId || companyId === 'your-valid-id') {
    console.error("Invalid or missing Company ID in localStorage:", companyId);
    showNoDeviceMessage = true;
    isLoading = false;
    return;
}

    const apiUrl = `${apiUrlBase}/getAll/${companyId}`;
    console.log("Fetching devices from:", apiUrl);

    try {
        const response = await fetch(apiUrl);
        if (!response.ok) {
            const errorText = await response.text();
            console.error("Response error text:", errorText);
            throw new Error(`Error: ${response.status}`);
        }

        const data = await response.json();
        if (data.error === "No devices found !" || data.length === 0) {
            showTable = false;
            showNoDeviceMessage = true;
        } else {
            devices = data;
            showTable = true;
            showNoDeviceMessage = false;
        }
    } catch (error) {
        console.error('Error fetching devices:', error);
    } finally {
        isLoading = false;
    }
}


    // @ts-ignore
    function accessKeyCreate(firstFourDigit, lastFourDigit) {
        const createUuidForAccessKey = uuidv4().replace(/-/g, '').substring(0, 6);
        return firstFourDigit + createUuidForAccessKey + lastFourDigit;
    }

    async function addData() {
        showNoDeviceMessage = false;
        isLoading = true;
        
        const accesskeyvalFirstDigit = Math.random().toString(36).substring(2, 6);
        const accesskeyvalLastDigit = Math.random().toString(36).substring(2, 6);
        const accessKey = accessKeyCreate(accesskeyvalFirstDigit, accesskeyvalLastDigit);

        const newDevice = {
            TimeZone: "Not Registered",
            DeviceID: "Not Registered",
            CID: localStorage.getItem('companyID'),
            DeviceName: "Not Registered",
            AccessKey: accessKey,
            AccessKeyGeneratedTime: new Date().toISOString(),
            LastModifiedBy: 'Admin'
        };

        try {
            const response = await fetch(`${apiUrlBase}/create`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(newDevice)
            });

            if (!response.ok) {
                const text = await response.text();
                throw new Error(`Error: ${response.status} - ${text}`);
            }

            await response.json();
            viewDevices();
        } catch (error) {
            console.error('Error adding device:', error);
        } finally {
            isLoading = false;
        }
    }

    // @ts-ignore
    function confirmDelete(accessKey) {
        deviceToDelete = accessKey;
        showDeleteModal = true;
    }
    

    async function deleteDevice() {
        const comId = localStorage.getItem('companyID');
        const apiUrl = `${apiUrlBase}/delete/${deviceToDelete}/${comId}/Admin`;

        try {
            const response = await fetch(apiUrl, {
                method: 'PUT'
            });

            if (!response.ok) {
                throw new Error(`Error: ${response.status}`);
            }

            const data = await response.json();
            if (data.error) {
                alert(data.error);
            } else {
                viewDevices();
            }
        } catch (error) {
            console.error('Error deleting device:', error);
        } finally {
            showDeleteModal = false;
            deviceToDelete = '';
        }
    }

</script>

<div class="min-h-screen h-[100vh] bg-gray-100 flex flex-col">
    <!-- Main Content -->
    <main class="flex-grow container mx-auto px-4 py-8 md:pl-72">
        <!-- Loading Overlay -->
        <!-- {#if isLoading}
            <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
                <div class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-blue-900"></div>
            </div>
        {/if} -->

        <!-- Device Table -->
        {#if showTable}
            <div class="bg-white rounded-xl shadow-md overflow-hidden mb-8">
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th scope="col" class="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Time zone</th>
                                <th scope="col" class="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Access Key</th>
                                <th scope="col" class="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Device Id</th>
                                <th scope="col" class="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Device Name</th>
                                <th scope="col" class="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Action</th>
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200">
                            {#each devices as device}
                                <tr>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">{device.TimeZone}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">
                                        {maskString(device.AccessKey, 4)}
                                        <button 
                                            class="ml-2 text-blue-900 hover:text-blue-700"
                                            on:click={() => copyAccessKey(device.AccessKey)}
                                        >
                                            <i class="far fa-copy"></i>
                                        </button>
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">{device.DeviceID}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">{device.DeviceName}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">
                                        <button 
                                            class="text-blue-900 hover:text-red-700"
                                            on:click={() => confirmDelete(device.AccessKey)}
                                        >
                                            <i class="fas fa-trash"></i>
                                        </button>
                                    </td>
                                </tr>
                            {/each}
                        </tbody>
                    </table>
                </div>
            </div>
        {/if}

        <!-- No Devices Message -->
        {#if showNoDeviceMessage}
            <div class="text-center py-16">
                <p class="text-gray-600 mb-6">No device Added</p>
                <button 
                    class="border border-blue-900 text-blue-900 bg-white px-6 py-2 rounded-md hover:bg-blue-900 hover:text-white transition-colors"
                    on:click={addData}
                >
                    Add device
                </button>
            </div>
        {/if}
    </main>

    <!-- Modals -->
    <!-- Home Page Confirmation Modal -->
    {#if showHomeModal}
        <div class="fixed inset-0 flex items-center justify-center z-50 transition-opacity duration-300"
        style="background: rgba(0, 0, 0, 0.5)">
            <div class="bg-white rounded-lg shadow-xl max-w-md w-full transform transition-all duration-300">
                <div class="bg-blue-900 text-white px-6 py-4 rounded-t-lg">
                    <h3 class="text-lg font-semibold text-center">Home</h3>
                </div>
                <div class="p-6">
                    <h5 class="font-bold mb-6 text-center">Are you sure you want to go home?</h5>
                    <div class="flex justify-center space-x-4">
                        <button 
                            class="bg-blue-900 text-white px-6 py-2 rounded-md hover:bg-blue-800 transition-colors"
                            on:click={confirmHomeNavigation}
                        >
                            Yes
                        </button>
                        <button 
                            class="border border-blue-900 text-blue-900 px-6 py-2 rounded-md hover:bg-gray-100 transition-colors"
                            on:click={() => showHomeModal = false}
                        >
                            No
                        </button>
                    </div>
                </div>
            </div>
        </div>
    {/if}

    <!-- Delete Device Confirmation Modal -->
    {#if showDeleteModal}
        <div class="fixed inset-0 flex items-center justify-center z-50 transition-opacity duration-300"
        style="background: rgba(0, 0, 0, 0.5)">
            <div class="bg-white rounded-lg shadow-xl max-w-md w-full transform transition-all duration-300">
                <div class="bg-red-600 text-white px-6 py-4 rounded-t-lg">
                    <h3 class="text-lg font-semibold text-center">Delete Device</h3>
                </div>
                <div class="p-6">
                    <h5 class="font-bold mb-6 text-center">Are you sure you want to delete this device?</h5>
                    <div class="flex justify-center space-x-4">
                        <button 
                            class="bg-red-600 text-white px-6 py-2 rounded-md hover:bg-red-500 transition-colors"
                            on:click={deleteDevice}
                        >
                            Yes
                        </button>
                        <button 
                            class="border border-red-600 text-red-600 px-6 py-2 rounded-md hover:bg-gray-100 transition-colors"
                            on:click={() => showDeleteModal = false}
                        >
                            No
                        </button>
                    </div>
                </div>
            </div>
        </div>
    {/if}
</div>