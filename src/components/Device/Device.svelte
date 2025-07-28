<script lang="ts">
    import { onMount } from 'svelte';
    import { v4 as uuidv4 } from 'uuid';
  
    // Constants
    const API_URL_BASE = "https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/device";
    const LOCAL_STORAGE_KEYS = {
      USERNAME: "username",
      COMPANY_ID: "companyID",
      CUSTOM_ID: "customId",
      PASSWORD: "password"
    } as const;
  
    // UI States
    let showTable = false;
    let showNoDeviceMessage = false;
    let isLoading = false;
    
    // Modal States
    let showDeleteModal = false;
    let showCopyTooltip = false;
    
    // Data
    let devices: Device[] = [];
    let deviceToDelete = '';
    let copiedAccessKey = '';
    let errorMessage = '';
  
    // Type Definitions
    type Device = {
      TimeZone: string;
      DeviceID: string;
      CID: string;
      DeviceName: string;
      AccessKey: string;
      AccessKeyGeneratedTime: string;
      LastModifiedBy: string;
    };
  
    // Initialize on component mount
    onMount(() => {
      // Handles clicks outside of modals or sidebars (currently a placeholder)
      function handleOutsideClick(event: MouseEvent) {
        // Implement logic if needed, or leave empty if not required
      }

      const init = async () => {
        try {

        await viewDevices();
      
          document.addEventListener('click', handleOutsideClick);
        } catch (error) {
          console.error('Initialization error:', error);
          errorMessage = 'Failed to initialize device data';
        }
      };
      init();
    
      return () => {
        document.removeEventListener('click', handleOutsideClick);
      };
    });
  
    
  
    // Device Management Functions
    const maskString = (input: string, visibleChars = 4): string => {
      if (!input) return '';
      visibleChars = Math.min(Math.max(visibleChars, 0), input.length);
      return '*'.repeat(input.length - visibleChars) + input.slice(-visibleChars);
    };
  
    const copyAccessKey = async (accessKey: string) => {
      try {
        await navigator.clipboard.writeText(accessKey);
        copiedAccessKey = accessKey;
        showCopyTooltip = true;
        setTimeout(() => showCopyTooltip = false, 2000);
      } catch (err) {
        console.error('Failed to copy:', err);
        errorMessage = 'Failed to copy access key';
      }
    };
  
    const viewDevices = async (): Promise<void> => {
      isLoading = true;
      errorMessage = '';
      const companyId = localStorage.getItem(LOCAL_STORAGE_KEYS.COMPANY_ID);
  
      if (!companyId) {
        errorMessage = "Company ID not found";
        showNoDeviceMessage = true;
        isLoading = false;
        return;
      }
  
      try {
        const response = await fetch(`${API_URL_BASE}/getAll/${companyId}`);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
  
        const data = await response.json();
        console.log('viewDevices API response:', data);
        localStorage.setItem('deviceData', JSON.stringify(data));

        if (data.error || data.length === 0) {
          console.log('No devices found or error:', data.error);
          showTable = false;
          showNoDeviceMessage = true;
          devices = [];
          if (data.error) errorMessage = data.error;
        } else {
          devices = Array.isArray(data) ? data : [data];
          console.log('Updated devices array:', devices.length, 'devices');
          showTable = true;
          showNoDeviceMessage = false;
        }
      } catch (error) {
        console.error('Error fetching devices:', error);
        errorMessage = 'Failed to load devices';
        showNoDeviceMessage = true;
      } finally {
        isLoading = false;
      }
    };
  
    const generateRandomString = (length = 4): string => {
      return Math.random().toString(36).substring(2, 2 + length).padEnd(length, '0');
    };
  
    const createAccessKey = (): string => {
      return `${generateRandomString(4)}${uuidv4().replace(/-/g, '').substring(0, 6)}${generateRandomString(4)}`;
    };
  
    const addDevice = async (): Promise<void> => {
      isLoading = true;
      errorMessage = '';
      showNoDeviceMessage = false;
      showTable=true;
      
      const companyId = localStorage.getItem(LOCAL_STORAGE_KEYS.COMPANY_ID);
      if (!companyId) {
        errorMessage = "Company ID not found";
        isLoading = false;
        return;
      }
  
      const newDevice: Device = {
        TimeZone: "Not Registered",
        DeviceID: "Not Registered",
        CID: companyId,
        DeviceName: "Not Registered",
        AccessKey: createAccessKey(),
        AccessKeyGeneratedTime: new Date().toISOString(),
        LastModifiedBy: 'Admin'
      };
      console.log(newDevice.AccessKey)
  
      try {
        const response = await fetch(`${API_URL_BASE}/create`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
          },
          body: JSON.stringify(newDevice)
        });
  
        if (!response.ok) {
          const errorData = await response.json().catch(() => ({}));
          throw new Error(errorData.message || `HTTP error! status: ${response.status}`);
        }
  
        await viewDevices();
      } catch (error) {
        console.error('Error adding device:', error);
        errorMessage = error instanceof Error ? error.message : 'Failed to add device';
      } finally {
        isLoading = false;
      }
    };
  
    const confirmDelete = (accessKey: string) => {
      deviceToDelete = accessKey;
      showDeleteModal = true;
    };
  
    const deleteDevice = async (): Promise<void> => {
      if (!deviceToDelete) return;
      
      isLoading = true;
      errorMessage = '';
      const companyId = localStorage.getItem(LOCAL_STORAGE_KEYS.COMPANY_ID);
      
      if (!companyId) {
        errorMessage = "Company ID not found";
        isLoading = false;
        return;
      }
  
      try {
        console.log('Deleting device with AccessKey:', deviceToDelete);
        const response = await fetch(`${API_URL_BASE}/delete/${deviceToDelete}/${companyId}/Admin`, {
          method: 'PUT',
          headers: {
            'Accept': 'application/json'
          }
        });
  
        if (!response.ok) {
          const errorData = await response.json().catch(() => ({}));
          throw new Error(errorData.message || `HTTP error! status: ${response.status}`);
        }
  
        // Optimistically remove device from local array first
        devices = devices.filter(device => device.AccessKey !== deviceToDelete);
        console.log('Optimistically removed device, remaining:', devices.length);
        
        // Then refresh from server to ensure consistency
        await viewDevices();
      } catch (error) {
        console.error('Error deleting device:', error);
        errorMessage = error instanceof Error ? error.message : 'Failed to delete device';
        // Refresh devices on error to restore correct state
        await viewDevices();
      } finally {
        isLoading = false;
        showDeleteModal = false;
        deviceToDelete = '';
      }
    };
  

  </script>

<div class=" bg-gray-100 flex flex-col pt-28 pb-19 ">
    <!-- Main Content -->
    <main class="flex-grow container mx-auto px-4 py-8 ">
        <!-- Loading Overlay -->
        {#if isLoading}
            <div class="fixed inset-0 flex items-center justify-center z-50"
            style="background: rgba(0, 0, 0, 0.5)">
                <div class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"></div>
            </div>
        {/if}

        <!-- Device Table -->
        {#if devices.length != 0}
        <div class="max-w-7xl flex justify-end py-5">
          <button 
                    class="border border-[#02066F] text-[#02066F] bg-white px-6 py-2 rounded-md transition-colors cursor-pointer"
                    on:click={addDevice}
                >
                    Add device
                </button> 
        </div>
            <div class="max-w-5xl mx-auto bg-white rounded-xl  overflow-hidden mb-8 border-1 border-gray-300">
              
              <div class="overflow-x-auto">
                  
                    <table class="w-full divide-y divide-gray-200">
                        <thead>
                            <tr>
                                <th scope="col" class="px-6 py-3 text-center text-base font-bold tracking-wider">Time zone</th>
                                <th scope="col" class="px-6 py-3 text-center text-base font-bold tracking-wider">Access Key</th>
                                <th scope="col" class="px-6 py-3 text-center text-base font-bold tracking-wider">Device Id</th>
                                <th scope="col" class="px-6 py-3 text-center text-base font-bold tracking-wider">Device Name</th>
                                <th scope="col" class="px-6 py-3 text-center text-base font-bold tracking-wider">Action</th>
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200">
                            {#each devices as device}
                                <tr>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center">{device.TimeZone}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center">
                                        {maskString(device.AccessKey, 4)}
                                        <button 
                                            class="ml-2 text-[#02066F] hover:text-black cursor-pointer"
                                            on:click={() => copyAccessKey(device.AccessKey)}
                                        >
                                            <i class="far fa-copy"></i>
                                        </button>
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center">{device.DeviceID}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center">{device.DeviceName}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center">
                                        <button 
                                            class="text-[#02066F] p-1 cursor-pointer"
                                            on:click={() => confirmDelete(device.AccessKey)}
                                        >
                                            <i class="fas fa-trash "></i>
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
            <div class="text-center h-[112px]">
                <p class="text-gray-600 mb-[17px]">No device Added</p>
                <button 
                    class="border border-[#02066F] text-[#02066F] bg-white px-6 py-2 rounded-md transition-colors cursor-pointer"
                    on:click={addDevice}
                >
                    Add device
                </button> 
            </div>
        {/if}
    </main>

    <!-- Modals -->

    <!-- Delete Device Confirmation Modal -->
    {#if showDeleteModal}
        <div class="fixed inset-0 flex items-center justify-center z-50 transition-opacity duration-300"
        style="background: rgba(0, 0, 0, 0.5)">
            <div class="bg-white rounded-md max-w-sm w-full shadow-xl">
                <div class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center text-center">
                    <h3 class="text-xl font-semibold p-2 text-white pl-30">Delete</h3>
                    <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2"  on:click={() => showDeleteModal = false}>
                      ×</button>
                </div>
              
                <div class="p-4">
                    <h5 class="font-bold text-md mb-6 text-center">Are you sure, you want to remove the device?</h5>
                    <div class="flex justify-center space-x-4">
                        <button 
                            class="bg-[#02066F] opacity-80 cursor-pointer text-white px-6 py-2 rounded-md hover:opacity-70 transition-colors"
                            on:click={deleteDevice}
                        >
                            Yes
                        </button>
                        <button 
                            class="border border-[#02066F] text-blue-800 px-6 py-2 rounded-md cursor-pointer hover:bg-gray-100 transition-colors"
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