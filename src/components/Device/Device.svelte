<script lang="ts">
  import { onMount } from "svelte";
  const browser = typeof window !== "undefined";
  import { v4 as uuidv4 } from "uuid";

  // Constants
  const API_URL_BASE =
    "https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/device";
  const LOCAL_STORAGE_KEYS = {
    USERNAME: "username",
    COMPANY_ID: "companyID",
    CUSTOM_ID: "customId",
    PASSWORD: "password",
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
  let deviceToDelete = "";
  let copiedAccessKey = "";
  let errorMessage = "";
  let maxDevices = 0;
  let editingDevice = "";
  let centerNativeValues: { [key: string]: string } = {};

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
    if (!browser) return;
    
    // Initialize maxDevices from localStorage
    const limitStr = localStorage.getItem("NoOfDevices") || "";
    maxDevices = parseInt(limitStr, 10) || 0;
    
    // Load center native values from localStorage
    const savedValues = localStorage.getItem("centerNativeValues");
    if (savedValues) {
      centerNativeValues = JSON.parse(savedValues);
    }

    // Handles clicks outside of modals or sidebars (currently a placeholder)
    function handleOutsideClick(event: MouseEvent) {
      // Implement logic if needed, or leave empty if not required
    }

    const init = async () => {
      try {
        await viewDevices();

        document.addEventListener("click", handleOutsideClick);
      } catch (error) {
        console.error("Initialization error:", error);
        errorMessage = "Failed to initialize device data";
      }
    };
    init();

    return () => {
      document.removeEventListener("click", handleOutsideClick);
    };
  });

  // Device Management Functions
  const maskString = (input: string, visibleChars = 4): string => {
    if (!input) return "";
    visibleChars = Math.min(Math.max(visibleChars, 0), input.length);
    return "*".repeat(input.length - visibleChars) + input.slice(-visibleChars);
  };

  const saveCenterNativeValues = () => {
    if (browser) {
      localStorage.setItem("centerNativeValues", JSON.stringify(centerNativeValues));
    }
  };

  const handleCenterNativeEdit = (accessKey: string) => {
    editingDevice = accessKey;
  };

  const handleCenterNativeKeydown = (event: KeyboardEvent, accessKey: string) => {
    if (event.key === 'Enter') {
      editingDevice = "";
      saveCenterNativeValues();
    }
  };

  const copyAccessKey = async (accessKey: string) => {
    if (!browser) return;
    try {
      await navigator.clipboard.writeText(accessKey);
      copiedAccessKey = accessKey;
      showCopyTooltip = true;
      setTimeout(() => (showCopyTooltip = false), 2000);
    } catch (err) {
      console.error("Failed to copy:", err);
      errorMessage = "Failed to copy access key";
    }
  };

  const viewDevices = async (): Promise<void> => {
    if (!browser) return;
    isLoading = true;
    errorMessage = "";
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
      console.log("viewDevices API response:", data);
      if (browser) localStorage.setItem("deviceData", JSON.stringify(data));

      if (data.error || data.length === 0) {
        console.log("No devices found or error:", data.error);
        showTable = false;
        showNoDeviceMessage = true;
        devices = [];
        if (data.error) errorMessage = data.error;
      } else {
        devices = Array.isArray(data) ? data : [data];
        console.log("Updated devices array:", devices.length, "devices");
        showTable = true;
        showNoDeviceMessage = false;
      }
    } catch (error) {
      console.error("Error fetching devices:", error);
      errorMessage = "Failed to load devices";
      showNoDeviceMessage = true;
    } finally {
      isLoading = false;
    }
  };

  const generateRandomString = (length = 4): string => {
    return Math.random()
      .toString(36)
      .substring(2, 2 + length)
      .padEnd(length, "0");
  };

  const createAccessKey = (): string => {
    return `${generateRandomString(4)}${uuidv4().replace(/-/g, "").substring(0, 6)}${generateRandomString(4)}`;
  };

  const addDevice = async (): Promise<void> => {
    if (!browser) return;
    isLoading = true;
    errorMessage = "";
    showNoDeviceMessage = false;
    showTable = true;

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
      LastModifiedBy: "Admin",
    };

    try {
      const response = await fetch(`${API_URL_BASE}/create`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Accept: "application/json",
        },
        body: JSON.stringify(newDevice),
      });

      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}));
        throw new Error(
          errorData.message || `HTTP error! status: ${response.status}`,
        );
      }

      await viewDevices();
    } catch (error) {
      console.error("Error adding device:", error);
      errorMessage =
        error instanceof Error ? error.message : "Failed to add device";
    } finally {
      isLoading = false;
    }
  };

  const confirmDelete = (accessKey: string) => {
    deviceToDelete = accessKey;
    showDeleteModal = true;
  };

  const deleteDevice = async (): Promise<void> => {
    if (!browser || !deviceToDelete) return;

    showDeleteModal = false;
    isLoading = true;
    errorMessage = "";
    const companyId = localStorage.getItem(LOCAL_STORAGE_KEYS.COMPANY_ID);

    if (!companyId) {
      errorMessage = "Company ID not found";
      isLoading = false;
      return;
    }

    try {
      console.log("Deleting device with AccessKey:", deviceToDelete);
      const response = await fetch(
        `${API_URL_BASE}/delete/${deviceToDelete}/${companyId}/Admin`,
        {
          method: "PUT",
          headers: {
            Accept: "application/json",
          },
        },
      );

      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}));
        throw new Error(
          errorData.message || `HTTP error! status: ${response.status}`,
        );
      }

      // Optimistically remove device from local array first
      devices = devices.filter((device) => device.AccessKey !== deviceToDelete);
      console.log("Optimistically removed device, remaining:", devices.length);

      // Then refresh from server to ensure consistency
      await viewDevices();
    } catch (error) {
      console.error("Error deleting device:", error);
      errorMessage =
        error instanceof Error ? error.message : "Failed to delete device";
      // Refresh devices on error to restore correct state
      await viewDevices();
    } finally {
      isLoading = false;
      showDeleteModal = false;
      deviceToDelete = "";
    }
  };
</script>

<div class=" bg-gray-100 flex flex-col pt-28 pb-19">
  <!-- Main Content -->
  <main class="flex-grow container mx-auto px-4 py-8">
    <!-- Loading Overlay -->
    {#if isLoading}
      <div
        class="fixed inset-0 flex items-center justify-center z-50"
        style="background: rgba(0, 0, 0, 0.5)"
      >
        <div
          class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"
        ></div>
      </div>
    {/if}

    <!-- Device Table -->
    {#if devices.length != 0}
      <div
        class="relative group max-w-7xl flex justify-end px-0 md:px-4 xl:px-6 py-5"
      >
        <button
          class="border border-[#02066F] text-[#02066F] bg-white px-6 py-2 rounded-md transition-colors cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
          on:click={addDevice}
          disabled={devices.length >= maxDevices}
        >
          Add device
        </button>

        
      </div>

      <div class="max-w-5xl mx-auto bg-white rounded-xl overflow-hidden mb-8 border-1 border-gray-300">
        <div class="overflow-x-auto">
          <table class="w-full divide-y divide-gray-200">
            <thead>
              <tr>
                <th
                  scope="col"
                  class="px-6 py-3 text-center text-base font-bold tracking-wider"
                  >Time zone</th
                >
                <th
                  scope="col"
                  class="px-6 py-3 text-center text-base font-bold tracking-wider"
                  >Access Key</th
                >
                <th
                  scope="col"
                  class="px-6 py-3 text-center text-base font-bold tracking-wider"
                  >Device Id</th
                >
                <th
                  scope="col"
                  class="px-6 py-3 text-center text-base font-bold tracking-wider"
                  >Device Name</th
                >
                <th
                  scope="col"
                  class="px-6 py-3 text-center text-base font-bold tracking-wider
                  ">Center Native</th>
                <th
                  scope="col"
                  class="px-6 py-3 text-center text-base font-bold tracking-wider"
                  >Action</th
                >
              </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-200">
              {#each devices as device}
                <tr>
                  <td
                    class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                    >{device.TimeZone}</td
                  >
                  <td
                    class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                  >
                    {maskString(device.AccessKey, 4)}
                    <button
                      class="ml-2 text-[#02066F] hover:text-black cursor-pointer"
                      on:click={() => copyAccessKey(device.AccessKey)}
                    >
                      <i class="far fa-copy"></i>
                    </button>
                  </td>
                  <td
                    class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                    >{device.DeviceID}</td
                  >
                  <td
                    class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                    >{device.DeviceName}</td
                  >
                  <td
                    class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                    on:click={() => handleCenterNativeEdit(device.AccessKey)}
                  >
                    {#if editingDevice === device.AccessKey}
                      <input 
                        type="text" 
                        bind:value={centerNativeValues[device.AccessKey]} 
                        on:keydown={(e) => handleCenterNativeKeydown(e, device.AccessKey)}
                        on:blur={() => editingDevice = ''}
                        class="w-full text-center border-1 border-gray-400 focus:outline-none" 
                        autofocus
                      />
                    {:else}
                      <span class="cursor-pointer">
                        {#if centerNativeValues[device.AccessKey]}
                          {centerNativeValues[device.AccessKey]}
                        {:else}
                          <i class="fas fa-pencil-alt"></i>
                        {/if}
                      </span>
                    {/if}
                  </td>
                  <td
                    class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                  >
                    <button
                      class="text-[#02066F] p-1 cursor-pointer"
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

      {#if devices.length >= maxDevices}
          <div>
            <p class="text-center text-[#02066F] font-semibold">
              You have reached the device registration limit. If you need to add
              more devices, please <a
                href="/contact"
                class="text-yellow-700 hover:underline">contact!</a
              >
            </p>
          </div>
          {:else}
          <p class="text-center text-[#02066F] font-semibold">You can add up to {maxDevices} devices.</p>
        {/if}
      
    {/if}

    <!-- No Devices Message -->
    {#if showNoDeviceMessage}
      <div class="text-center h-[112px]">
        <p class="text-gray-600 mb-[20px]">No device Added</p>
        <button
          class="border border-[#02066F] text-[#02066F] bg-white px-6 py-2 rounded-md transition-colors cursor-pointer "
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
    <div
      class="fixed inset-0 flex items-center justify-center z-50 transition-opacity duration-300"
      style="background: rgba(0, 0, 0, 0.5)"
    >
      <div class="bg-white rounded-md max-w-sm w-full shadow-xl">
        <div
          class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center text-center"
        >
          <h3 class="text-xl font-semibold p-2 text-white pl-30">Delete</h3>
          <button
            class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2"
            on:click={() => (showDeleteModal = false)}
          >
            ×</button
          >
        </div>

        <div class="p-4">
          <h5 class="font-bold text-md mb-6 text-center">
            Are you sure, you want to remove the device?
          </h5>
          <div class="flex justify-center space-x-4">
            <button
              class="bg-[#02066F] opacity-80 cursor-pointer text-white px-6 py-2 rounded-md hover:opacity-70 transition-colors"
              on:click={deleteDevice}
            >
              Yes
            </button>
            <button
              class="border border-[#02066F] text-blue-800 px-6 py-2 rounded-md cursor-pointer hover:bg-gray-100 transition-colors"
              on:click={() => (showDeleteModal = false)}
            >
              No
            </button>
          </div>
        </div>
      </div>
    </div>
  {/if}
</div>