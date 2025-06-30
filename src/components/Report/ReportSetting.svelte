<script lang="ts">
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';

    type ReportFrequency = 'Daily' | 'Weekly' | 'Biweekly' | 'Monthly' | 'Bimonthly';
    type ReportSetting = {
        CompanyReporterEmail: string;
        CID: string;
        IsDailyReportActive: boolean;
        IsWeeklyReportActive: boolean;
        IsBiWeeklyReportActive: boolean;
        IsMonthlyReportActive: boolean;
        IsBiMonthlyReportActive: boolean;
        LastModifiedBy: string;
    };

    const apiUrlBase = 'https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/company-report-type';
    const adminReportApi = 'https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/admin-report-type';

    let isLoading = false;
    let emailSettings: ReportSetting[] = [];
    let viewSetting: ReportFrequency = 'Weekly';
    let showAddModal = false;
    let showEditModal = false;
    let showViewEditModal = false;
    let showDeleteModal = false;
    let currentEmail = '';
    let currentFrequencies: ReportFrequency[] = [];
    let newEmail = '';
    let newFrequencies: ReportFrequency[] = [];
    let emailError = '';
    let frequencyError = '';
    let viewFrequencyError = '';
    // let showFrequencyDropdown = false;

    onMount(async () => {
        await loadReportSettings();
        loadViewSetting();
    });

    async function loadReportSettings() {
        isLoading = true;
        const company_id = localStorage.getItem('companyID') || '';
        
        try {
            const response = await fetch(`${apiUrlBase}/getAllReportEmail/${company_id}`);
            if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
            emailSettings = await response.json();
        } catch (error) {
            console.error('Failed to load report settings:', error);
            emailSettings = [];
        } finally {
            isLoading = false;
        }
    }

    function loadViewSetting() {
        const savedSetting = localStorage.getItem('reportType');
        if (savedSetting && ['Daily', 'Weekly', 'Biweekly', 'Monthly', 'Bimonthly'].includes(savedSetting)) {
            viewSetting = savedSetting as ReportFrequency;
        }
    }

    function openAddModal() {
        newEmail = '';
        newFrequencies = [];
        emailError = '';
        frequencyError = '';
        showFrequencyDropdown = false; // Initially hide dropdown
        showAddModal = true;
    }
   
    
    function openEditModal(email: string, frequencies: ReportFrequency[]) {
        currentEmail = email;
        currentFrequencies = [...frequencies];
        emailError = '';
        frequencyError = '';
        showEditModal = true;
    }

    function closeAddModal() {
        showAddModal = false;
        newEmail = '';
        newFrequencies = [];
        emailError = '';
        frequencyError = '';
        showFrequencyDropdown = false; // Reset dropdown state
    }

    function openViewEditModal() {
        showViewEditModal = true;
    }

    function openDeleteModal(email: string) {
        currentEmail = email;
        showDeleteModal = true;
    }

    function validateEmail(email: string): boolean {
        const isEmail = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
        
        if (!email.trim()) {
            emailError = 'Email is required';
            return false;
        } else if (!isEmail.test(email)) {
            emailError = 'Email pattern is invalid';
            return false;
        }
        
        emailError = '';
        return true;
    }

    function validateFrequencies(frequencies: ReportFrequency[]): boolean {
        if (frequencies.length === 0) {
            frequencyError = 'Please select at least one option';
            return false;
        } else if (frequencies.length > 2) {
            frequencyError = 'Please select maximum two options';
            return false;
        }
        
        frequencyError = '';
        return true;
    }

    function validateViewFrequency(frequency: ReportFrequency): boolean {
        if (!frequency) {
            viewFrequencyError = 'Please select an option';
            return false;
        }
        
        viewFrequencyError = '';
        return true;
    }

    // function toggleFrequency(frequency: ReportFrequency, list: ReportFrequency[]) {
    //     if (list.includes(frequency)) {
    //         return list.filter(f => f !== frequency);
    //     } else {
    //         return [...list, frequency];
    //     }
    // }

    // async function saveReportSettings() {
    //     if (!validateEmail(newEmail)) return;
    //     if (!validateFrequencies(newFrequencies)) return;
        
    //     const company_id = localStorage.getItem('companyID') || '';
    //     const setting: ReportSetting = {
    //         CompanyReporterEmail: newEmail,
    //         CID: company_id,
    //         IsDailyReportActive: newFrequencies.includes('Daily'),
    //         IsWeeklyReportActive: newFrequencies.includes('Weekly'),
    //         IsBiWeeklyReportActive: newFrequencies.includes('Biweekly'),
    //         IsMonthlyReportActive: newFrequencies.includes('Monthly'),
    //         IsBiMonthlyReportActive: newFrequencies.includes('Bimonthly'),
    //         LastModifiedBy: 'Admin'
    //     };

    //     // isLoading = true;
    //     try {
    //         const response = await fetch(`${apiUrlBase}/create`, {
    //             method: 'POST',
    //             headers: { 'Content-Type': 'application/json' },
    //             body: JSON.stringify(setting)
    //         });

    //         if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
            
    //         showAddModal = false;
    //         await loadReportSettings();
    //     } catch (error) {
    //         console.error('Failed to save settings:', error);
    //     } finally {
    //         isLoading = false;
    //     }
    //     // console.log('Payload:', yourPayloadObject);
    // }

    async function saveReportSettings() {
    if (!validateEmail(newEmail)) return;
    if (!validateFrequencies(newFrequencies)) return;

    const company_id = localStorage.getItem('companyID') || '';
    if (!company_id) {
        console.error("Missing company ID in localStorage");
        return;
    }

    const setting: ReportSetting = {
        CompanyReporterEmail: newEmail,
        CID: company_id,
        IsDailyReportActive: newFrequencies.includes('Daily'),
        IsWeeklyReportActive: newFrequencies.includes('Weekly'),
        IsBiWeeklyReportActive: newFrequencies.includes('Biweekly'),
        IsMonthlyReportActive: newFrequencies.includes('Monthly'),
        IsBiMonthlyReportActive: newFrequencies.includes('Bimonthly'),
        LastModifiedBy: 'Admin'
    };

    console.log("Sending to API:", setting); // Debugging

    try {
        const response = await fetch(`${apiUrlBase}/create`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(setting)
        });

        if (!response.ok) {
            const errorText = await response.text();
            throw new Error(`HTTP error ${response.status}: ${errorText}`);
        }

        showAddModal = false;
        await loadReportSettings();
    } catch (error) {
        console.error('Failed to save settings:', error);
        if (error instanceof Error) {
          console.error(`Failed to save settings: ${error.message}`);
        } else {
           console.error('Failed to save settings: An unknown error occurred.');
        }
    } finally {
        isLoading = false;
    }
}



    async function updateReportSettings() {
        if (!validateEmail(currentEmail)) return;
        if (!validateFrequencies(currentFrequencies)) return;
        
        const company_id = localStorage.getItem('companyID') || '';
        const setting: ReportSetting = {
            CompanyReporterEmail: currentEmail,
            CID: company_id,
            IsDailyReportActive: currentFrequencies.includes('Daily'),
            IsWeeklyReportActive: currentFrequencies.includes('Weekly'),
            IsBiWeeklyReportActive: currentFrequencies.includes('Biweekly'),
            IsMonthlyReportActive: currentFrequencies.includes('Monthly'),
            IsBiMonthlyReportActive: currentFrequencies.includes('Bimonthly'),
            LastModifiedBy: 'Admin'
        };

        isLoading = true;
        try {
            const response = await fetch(`${apiUrlBase}/update/${currentEmail}/${company_id}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(setting)
            });

            if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
            
            showEditModal = false;
            await loadReportSettings();
        } catch (error) {
            console.error('Failed to update settings:', error);
        } finally {
            isLoading = false;
        }
    }

    async function updateViewSetting() {
        if (!validateViewFrequency(viewSetting)) return;
        
        const company_id = localStorage.getItem('companyID') || '';
        const setting = {
            CID: company_id,
            ReportType: viewSetting
        };

        isLoading = true;
        try {
            const response = await fetch(`${adminReportApi}/update/${company_id}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(setting)
            });

            if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
            
            localStorage.setItem('reportType', viewSetting);
            showViewEditModal = false;
        } catch (error) {
            console.error('Failed to update view setting:', error);
        } finally {
            isLoading = false;
        }
    }

    async function deleteReportSetting() {
        const company_id = localStorage.getItem('companyID') || '';
        
        isLoading = true;
        try {
            const response = await fetch(`${apiUrlBase}/delete/${currentEmail}/${company_id}/Admin`, {
                method: 'PUT'
            });

            if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
            
            showDeleteModal = false;
            await loadReportSettings();
        } catch (error) {
            console.error('Failed to delete setting:', error);
        } finally {
            isLoading = false;
        }
    }

    function formatFrequencies(setting: ReportSetting): string {
        const frequencies: ReportFrequency[] = [];
        if (setting.IsDailyReportActive) frequencies.push('Daily');
        if (setting.IsWeeklyReportActive) frequencies.push('Weekly');
        if (setting.IsBiWeeklyReportActive) frequencies.push('Biweekly');
        if (setting.IsMonthlyReportActive) frequencies.push('Monthly');
        if (setting.IsBiMonthlyReportActive) frequencies.push('Bimonthly');
        return frequencies.join(', ');
    }

    let showFrequencyDropdown = false;
    
    function selectFrequency(freq: ReportFrequency) {
        if (newFrequencies.includes(freq)) {
            newFrequencies = newFrequencies.filter(f => f !== freq);
        } else {
            if (newFrequencies.length < 2) {
                newFrequencies = [...newFrequencies, freq];
            }
        }
        frequencyError = '';
    }

    // import { onClickOutside } from './utils'; // Optional, for clicking outside to close dropdown

let frequencies = ['Daily','Weekly', 'Biweekly', 'Monthly', 'Bimonthly'];
let selectedFrequencies: string[] = [];
let showDropdown = false;
// let frequencyError = '';

function toggleDropdown() {
  showDropdown = !showDropdown;
}

function toggleFrequency(freq: string) {
  if (selectedFrequencies.includes(freq)) {
    selectedFrequencies = selectedFrequencies.filter(f => f !== freq);
  } else if (selectedFrequencies.length < 2) {
    selectedFrequencies = [...selectedFrequencies, freq];
    frequencyError = '';
  } else {
    frequencyError = 'You can select up to 2 frequencies only.';
  }
}

$: displayValue = selectedFrequencies.join(', ') || [...currentFrequencies];
</script>

<div class="min-h-screen bg-gray-100 p-4 md:p-8">

    <!-- Main Content -->
    <div class="max-w-5xl mx-auto pt-25">
        <!-- Report Email Settings Section -->
        <div class="flex flex-col md:flex-row md:items-center justify-between md:justify-between mb-6">
            <h2 class="text-xl md:text-2xl xl:text-3xl font-bold mb-4 md:mb-0 text-gray-800">Report Settings</h2>
            <button 
                on:click={openAddModal}
                class="px-4 py-2 text-[#02066F] border-1 border-[#02066F] cursor-pointer rounded-lg transition-colors duration-200 bg-white"
            >
                Add Entry
            </button>
        </div>
        <div class="bg-white rounded-xl shadow-md p-6 mb-8">

            <div class="overflow-x-auto">
                <table class="w-full divide-y divide-gray-200">
                    <thead class="">
                        <tr>
                            <th class="px-6 py-3 text-left text-base font-medium tracking-wider">Email</th>
                            <th class="px-6 py-3 text-left text-base font-medium tracking-wider">Frequency</th>
                            <th class="px-6 py-3 text-left text-base font-medium tracking-wider">Action</th>
                        </tr>
                    </thead>
                    <tbody class="bg-white divide-y divide-gray-200">
                        {#if emailSettings.length === 0}
                            <tr>
                                <td colspan="3" class="px-6 py-4 text-center text-gray-500">
                                    {isLoading ? 'Loading...' : 'No report settings found'}
                                </td>
                            </tr>
                        {:else}
                            {#each emailSettings as setting}
                                <tr class="hover:bg-gray-50">
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                                        {setting.CompanyReporterEmail}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                                        {formatFrequencies(setting)}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                                        <div class="flex space-x-2">
                                            <button
                                                on:click={() => openEditModal(
                                                    setting.CompanyReporterEmail,
                                                    formatFrequencies(setting).split(', ') as ReportFrequency[]
                                                )}
                                                class="text-[#02066F]"
                                            >
                                                <!-- <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" />
                                                </svg> -->
                                                <!-- <button on:click={openViewEditModal} class="text-[#02066F]  hover:text-[#02066F]"> -->
                                                    <i class="fas fa-pencil-alt cursor-pointer"></i>
                                                  <!-- </button> -->
                                            </button>
                                            <button
                                                on:click={() => openDeleteModal(setting.CompanyReporterEmail)}
                                                class="text-[#02066F]"
                                            >
                                                <!-- <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                                                </svg> -->
                                                <i class="fas fa-trash cursor-pointer"></i>
                                            </button>
                                        </div>
                                    </td>
                                </tr>
                            {/each}
                        {/if}
                    </tbody>
                </table>
            </div>
        </div>

        <!-- Report View Settings Section -->
        <div class="flex flex-col md:flex-row md:items-center justify-between md:justify-between mb-6">
        <h2 class="text-xl md:text-2xl xl:text-3xl font-bold mb-4 md:mb-0 text-gray-800">Report View Settings</h2>
        </div>
        <div class="bg-white rounded-xl shadow-md p-6 mb-0">
            
            <div class="overflow-x-auto items-center justify-center">
                <table class="w-full divide-y divide-gray-200">
                    <thead class="">
                        <tr>
                            <th class="px-6 py-3 text-left text-base font-medium tracking-wider">Frequency</th>
                            <th class="px-6 py-3 text-left text-base font-medium tracking-wider">Action</th>
                        </tr>
                    </thead>
                    <tbody class="bg-white divide-y divide-gray-200">
                        <tr class="hover:bg-gray-50">
                            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                                {viewSetting}
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap text-base text-gray-900">
                                <button on:click={openViewEditModal} class="text-[#02066F]  hover:text-[#02066F]">
                                    <i class="fas fa-pencil-alt cursor-pointer"></i>
                                  </button>
                                
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <!-- Add Report Setting Modal -->
    {#if showAddModal}
    <div 
        transition:fade
        class="fixed inset-0 flex items-center justify-center z-50 p-4"
        style="background: rgba(0, 0, 0, 0.5)"
    >
        <div class="bg-white rounded-md max-w-sm w-full shadow-xl">
            <div class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center text-center">
                <h3 class="text-2xl font-semibold p-2 text-white">Report Details</h3>
                <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2" on:click={closeAddModal}> ×</button>        
            </div>
            <div class="p-6 text-center">
                <!-- Email Input -->
                <div class="mb-4">
                    <input
                        type="email"
                        bind:value={newEmail}
                        class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg focus:outline-none text-center font-bold"
                        placeholder="Email"
                    />
                    {#if emailError}
                        <p class="mt-1 text-sm text-red-600">{emailError}</p>
                    {/if}
                </div>
                
                <!-- Frequency Dropdown -->
                <div class="mb-6">
                    <div class="relative">
                        <div 
                            class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg cursor-pointer flex justify-evenly items-center text-center"
                            on:click={() => showFrequencyDropdown = !showFrequencyDropdown}
                        >
                            <span class={newFrequencies.length === 0 ? 'text-gray-500' : 'text-[#02066F] text-base'}>
                                {newFrequencies.length === 0 ? 'Nothing selected' : newFrequencies.join(', ')}
                            </span>
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-gray-400" viewBox="0 0 20 20" fill="currentColor">
                                <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                            </svg>
                        </div>
                        
                        {#if showFrequencyDropdown}
                            <div class="absolute z-10 mt-1 w-full bg-white shadow-lg rounded-lg border border-gray-200 ">
                                {#each ['Daily', 'Weekly', 'Biweekly', 'Monthly', 'Bimonthly'] as freq}
                                    <div 
                                        class="px-4 py-3 hover:bg-gray-50 cursor-pointer flex items-center"
                                        on:click={() => selectFrequency(freq as ReportFrequency)}
                                    >
                                        <span class="text-[#02066F] text-base text-center">{freq}</span>
                                        {#if newFrequencies.includes(freq as ReportFrequency)}
                                            <!-- <span class="text-base text-[#02066F] ml-2">✔</span> -->
                                            <span class="text-xl font-bold text-[#02066F] ml-2">✓</span>
                                        {/if}
                                    </div>
                                {/each}
                            </div>
                        {/if}
                    </div>
                    {#if frequencyError}
                        <p class="mt-1 text-sm text-red-600">{frequencyError}</p>
                    {/if}
                </div>
                
                <!-- Save Button -->
                <button
                    on:click={saveReportSettings}
                    class="w-30 py-3 bg-[#02066F] cursor-pointer text-white rounded-lg text-center justify-center items-center hover:opacity-90"
                    disabled={!newEmail || newFrequencies.length === 0}
                >
                    Save
                </button>
            </div>
        </div>
    </div>
{/if}

    <!-- Edit Report Setting Modal -->
    {#if showEditModal}
        <div 
            transition:fade
            class="fixed inset-0 flex items-center justify-center z-50 p-4"
            style="background: rgba(0, 0, 0, 0.5)"
        >
            <div class="bg-white rounded-md max-w-sm w-full shadow-xl">
                <div class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center text-center">
                    <h3 class="text-2xl font-semibold p-2 text-white">Report Details</h3>
                    <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2"  on:click={() => showEditModal = false}>
                         ×</button>        
                </div>
                
                <div class="p-6 text-center">
                    <div class="mb-4">
                        <!-- <label for="edit-email" class="block text-sm font-medium text-gray-700 mb-1">Email</label> -->
                        <input
                            type="email"
                            id="edit-email"
                            bind:value={currentEmail}
                            class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg focus:outline-none text-center font-bold"
                            placeholder="Enter email"
                        />
                        {#if emailError}
                            <p class="mt-1 text-sm text-red-600">{emailError}</p>
                        {/if}
                    </div>
                    
                    <div class="mb-6">
                        <!-- <label class="block text-sm font-medium text-gray-700 mb-1">Frequency</label> -->
                        <div class="space-y-2">
                            <div
    class="border border-gray-300 rounded-md px-4 py-2 cursor-pointer bg-white"
    on:click={toggleDropdown}
  >
    {displayValue}
  </div>

  <!-- Dropdown options -->
  {#if showDropdown}
    <div class="dropdown">
      {#each frequencies as freq}
        <label class="flex items-center px-4 py-2 hover:bg-gray-100">
          <input
            type="checkbox"
            value={freq}
            checked={selectedFrequencies.includes(freq)}
            on:change={() => toggleFrequency(freq)}
            class="mr-2"
          />
          {freq}
        </label>
      {/each}
    </div>
  {/if}

                        </div>
                        {#if frequencyError}
                            <p class="mt-1 text-sm text-red-600">{frequencyError}</p>
                        {/if}
                    </div>
                    
                    <button
                    on:click={updateReportSettings}
                    class="w-30 py-3 bg-[#02066F] cursor-pointer text-white rounded-lg text-center justify-center items-center hover:opacity-90"
                    disabled={!newEmail || newFrequencies.length === 0}
                >
                    Save
                </button>
                    <!-- <div class="flex justify-center space-x-4">
                        <button
                            on:click={updateReportSettings}
                            class="px-6 py-2 bg-blue-900 text-white rounded-lg hover:bg-blue-800 transition-colors duration-200 disabled:opacity-50"
                            disabled={isLoading}
                        >
                            {isLoading ? 'Updating...' : 'Update'}
                        </button> -->
                        <!-- <button
                            on:click={() => showEditModal = false}
                            class="px-6 py-2 border border-blue-900 text-blue-900 rounded-lg hover:bg-gray-100 transition-colors duration-200"
                        >
                            Cancel
                        </button> -->
                    <!-- </div> -->
                </div>
            </div>
        </div>
    {/if}

    <!-- Edit View Setting Modal -->
    {#if showViewEditModal}
        <div 
            transition:fade
            class="fixed inset-0  flex items-center justify-center z-50 p-4"
            style="background: rgba(0, 0, 0, 0.5)"
        >
            <div class="bg-white rounded-xl max-w-xs w-full shadow-xl">
                <div class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center text-center">
                    <h3 class="text-2xl font-semibold p-2 text-white">Report Details</h3>
                    <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2" on:click={()=> showViewEditModal = false}> ×</button>        
                </div>
                
                <div class="p-6">
                    <div class="mb-6">
                        <!-- <label for="frequency" class="block text-sm font-medium text-gray-700 mb-1">Select Frequency</label> -->
                        <select
                          id="frequency"
                          bind:value={viewSetting}
                          class="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none "
                        >
                          <!-- <option value="">-- Select Frequency --</option> -->
                          <option value="Weekly">Weekly</option>
                          <option value="Biweekly">Biweekly</option>
                          <option value="Monthly">Monthly</option>
                          <option value="Bimonthly">Bimonthly</option>
                        </select>
                  
                        {#if viewFrequencyError}
                          <p class="mt-1 text-sm text-red-600">{viewFrequencyError}</p>
                        {/if}
                      </div>
                  
                    
                    <div class="flex justify-center space-x-4">
                        <button
                            on:click={updateViewSetting}
                            class="px-6 py-2 bg-[#02066F] text-white rounded-lg transition-colors duration-200 hover:opacity-90 cursor-pointer"
                            
                        >
                            Update
                        </button>
                    </div>
                </div>
            </div>
        </div>
    {/if}

    <!-- Delete Confirmation Modal -->
    {#if showDeleteModal}
        <div 
            transition:fade
            class="fixed inset-0  flex items-center justify-center z-50 p-4"
            style="background: rgba(0, 0, 0, 0.5)"
        >
            <div class="bg-white rounded-xl max-w-md w-full shadow-xl">

                <div class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center text-center">
                    <h3 class="text-2xl font-semibold p-2 text-white">Delete</h3>
                    <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2" on:click={()=> showDeleteModal = false}> ×</button>
                </div>

                
                <div class="p-4">
                    <p class="text-center text-lg font-bold mb-6 text-gray-800">Are you sure, you want to remove the employee?</p>
                    
                    <div class="flex justify-center space-x-4">
                        <button
                            on:click={deleteReportSetting}
                            class="px-6 py-2 bg-[#02066F] text-white rounded-lg transition-colors duration-200 hover:opacity-90 cursor-pointer"
                            disabled={isLoading}
                        >
                            {isLoading ? 'Deleting...' : 'Yes'}
                        </button>
                        <button
                            on:click={() => showDeleteModal = false}
                            class="px-6 py-2 border border-[#02066F] text-blue-900 rounded-lg hover:bg-gray-100 transition-colors cursor-pointer duration-200"
                        >
                            No
                        </button>
                    </div>
                </div>
            </div>
        </div>
    {/if}
</div>