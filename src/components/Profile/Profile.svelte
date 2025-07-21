<script lang="ts">
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';

    // State variables
    let isLoading = true;
    let companyEditMode = false;
    let customerEditMode = false;
    let showSuccessModal = false;
    let phoneError = '';
    let activeSection: 'company' | 'customer' | 'admin' = 'company';
    let userType: 'customer' | 'admin' | 'SuperAdmin' = 'customer';
    let passwordEditable = false;
    let showPassword = false;

    // API URLs
    const customerAPIUrlBase = `https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/customer`;
    const companyAPIUrlBase = `https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/company`;

    // Form data structure
    let formData = {
        companyName: '',
        username: '',
        password: '',
        decryptedPassword: '',
        companyStreet: '',
        companyCity: '',
        companyState: '',
        companyZip: '',
        firstName: '',
        lastName: '',
        customerStreet: '',
        customerCity: '',
        customerState: '',
        customerZip: '',
        email: '',
        phone: '',
        logo: '',
        adminPin: '',
        adminEmail: ''
    };

    let errors = {
    companyName: '',
    companyStreet: '',
    companyCity: '',
    companyState: '',
    companyZip: '',
    username: '',
    password: '',
    firstName: '',
    lastName: '',
    customerStreet: '',
    customerCity: '',
    customerState: '',
    customerZip: '',
    email: '',
    phone: ''
  };



    // Local storage data
    let companyId = '';
    let customerId = '';
    let adminEmail = '';

    
    onMount(async () => {
    companyId = localStorage.getItem('companyID') || '';
    customerId = localStorage.getItem('customerId') || '';
    adminEmail = localStorage.getItem('adminMail') || '';
    userType = localStorage.getItem('adminType') as 'customer' | 'admin' | 'SuperAdmin' || 'customer';

    // First load from localStorage
    const companyData = localStorage.getItem("companyProfileData");
    const customerData = localStorage.getItem("customerProfileData");
    const adminData = localStorage.getItem("adminProfileData");

    if (companyData) {
        const parsedCompany = JSON.parse(companyData);
        formData = {
            ...formData, // Keep existing structure
            ...parsedCompany, // Override with saved company data
            companyName: parsedCompany.CName,
            username: parsedCompany.UserName,
            password: parsedCompany.Password,
            companyStreet: parsedCompany.CAddress?.split('--')[0] || '',
            companyCity: parsedCompany.CAddress?.split('--')[1] || '',
            companyState: parsedCompany.CAddress?.split('--')[2] || '',
            companyZip: parsedCompany.CAddress?.split('--')[3] || '',
            logo: parsedCompany.CLogo || localStorage.getItem('companyLogo') || ''
        };
    }

    if (customerData) {
        const parsedCustomer = JSON.parse(customerData);
        formData = {
            ...formData,
            ...parsedCustomer,
            firstName: parsedCustomer.FName,
            lastName: parsedCustomer.LName,
            customerStreet: parsedCustomer.Address?.split('--')[0] || '',
            customerCity: parsedCustomer.Address?.split('--')[1] || '',
            customerState: parsedCustomer.Address?.split('--')[2] || '',
            customerZip: parsedCustomer.Address?.split('--')[3] || '',
            email: parsedCustomer.Email,
            phone: parsedCustomer.PhoneNumber ? formatPhoneNumberForDisplay(parsedCustomer.PhoneNumber) : ''
        };
    }

    isLoading = false;
});


 // Company validation
  function validateCompanyFields(): boolean {
    let isValid = true;
    
    // Reset company errors
    errors.companyName = '';
    errors.companyStreet = '';
    errors.companyCity = '';
    errors.companyState = '';
    errors.companyZip = '';
    errors.username = '';
    errors.password = '';

    // Validate each field
    if (!formData.companyName.trim()) {
      errors.companyName = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.companyStreet.trim()) {
      errors.companyStreet = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.companyCity.trim()) {
      errors.companyCity = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.companyState.trim()) {
      errors.companyState = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.companyZip.trim()) {
      errors.companyZip = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.username.trim()) {
      errors.username = 'Please fill out this field';
      isValid = false;
    }

    if (passwordEditable && !formData.password.trim()) {
      errors.password = 'Please fill out this field';
      isValid = false;
    }

    return isValid;
  }

  // Customer validation
  function validateCustomerFields(): boolean {
    let isValid = true;
    
    // Reset customer errors
    errors.firstName = '';
    errors.lastName = '';
    errors.customerStreet = '';
    errors.customerCity = '';
    errors.customerState = '';
    errors.customerZip = '';
    errors.email = '';
    errors.phone = '';

    // Validate each field
    if (!formData.firstName.trim()) {
      errors.firstName = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.lastName.trim()) {
      errors.lastName = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.customerStreet.trim()) {
      errors.customerStreet = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.customerCity.trim()) {
      errors.customerCity = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.customerState.trim()) {
      errors.customerState = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.customerZip.trim()) {
      errors.customerZip = 'Please fill out this field';
      isValid = false;
    }

    if (!formData.email.trim()) {
      errors.email = 'Please fill out this field';
      isValid = false;
    }

    if (formData.phone && !/^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$/.test(formData.phone)) {
      errors.phone = 'Please use format: (123) 456-7890';
      isValid = false;
    }

    return isValid;
  }

    // Encrypt functions
    async function encryptPasswordInput(): Promise<string> {
        const password = formData.decryptedPassword;
        const storedPassword = localStorage.getItem('passwordDecryptedValue');

        if (password === storedPassword) {
            return localStorage.getItem('password') || '';
        }

        const keyValue = new Uint8Array([16, 147, 220, 113, 166, 142, 22, 93, 241, 91, 13, 252, 112, 122, 119, 95]);
        return await encryptData(password, keyValue);
    }

    async function encryptData(data: string, key: Uint8Array): Promise<string> {
        const dataBuffer = new TextEncoder().encode(data);
        const algorithm = { name: 'AES-GCM', iv: generateRandomBytes(12) };

        const importedKey = await crypto.subtle.importKey(
            'raw', key, { name: 'AES-GCM' }, false, ['encrypt']
        );

        const encryptedData = await crypto.subtle.encrypt(
            algorithm, importedKey, dataBuffer
        );

        const iv = algorithm.iv;
        const encryptedDataWithIV = new Uint8Array(iv.byteLength + encryptedData.byteLength);
        encryptedDataWithIV.set(iv);
        encryptedDataWithIV.set(new Uint8Array(encryptedData), iv.byteLength);

        return btoa(String.fromCharCode(...new Uint8Array(encryptedDataWithIV)));
    }

    function generateRandomBytes(length: number): Uint8Array {
        const randomValues = new Uint8Array(length);
        crypto.getRandomValues(randomValues);
        return randomValues;
    }

    // API functions
    async function callCompanyAPI() {
        if (!companyId) return;

        const encryptedPassword = await encryptPasswordInput();
        const companyData = {
            CID: companyId,
            UserName: formData.username,
            CName: formData.companyName,
            CAddress: `${formData.companyStreet}--${formData.companyCity}--${formData.companyState}--${formData.companyZip}`,
            CLogo: formData.logo || localStorage.getItem("imageFile"),
            Password: encryptedPassword,
            ReportType: "Weekly",
            LastModifiedBy: 'Admin'
        };

        try {
            const response = await fetch(`${companyAPIUrlBase}/update/${companyId}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(companyData)
            });

            if (!response.ok) throw new Error(`Company update failed: ${response.status}`);

            localStorage.setItem("companyProfileData", JSON.stringify(companyData));
            showSuccessModal = true;
            setTimeout(() => showSuccessModal = false, 2000);
        } catch (error) {
            console.error("Company API Error:", error);
        }
    }

    async function callCustomerAPI() {
        if (!companyId || !customerId) return;

        const customerData = {
            CustomerID: customerId,
            CID: companyId,
            FName: formData.firstName,
            LName: formData.lastName,
            Address: `${formData.customerStreet}--${formData.customerCity}--${formData.customerState}--${formData.customerZip}`,
            PhoneNumber: formData.phone.replace(/\D/g, ''),
            Email: formData.email,
            IsActive: true,
            LastModifiedBy: 'Admin'
        };

        try {
            const response = await fetch(`${customerAPIUrlBase}/update/${customerId}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(customerData)
            });

            if (!response.ok) throw new Error(`Customer update failed: ${response.status}`);

            localStorage.setItem("customerProfileData", JSON.stringify(customerData));
            showSuccessModal = true;
            setTimeout(() => showSuccessModal = false, 2000);
        } catch (error) {
            console.error("Customer API Error:", error);
        }
    }



    // Phone formatting & validation
    function formatPhoneNumberForDisplay(phoneNumber: string): string {
        const cleaned = phoneNumber.replace(/\D/g, '');
        if (cleaned.length !== 10) return phoneNumber;

        const areaCode = cleaned.substring(0, 3);
        const centralOfficeCode = cleaned.substring(3, 6);
        const lineNumber = cleaned.substring(6);

        return `(${areaCode}) ${centralOfficeCode}-${lineNumber}`;
    }

    function formatPhoneNumberInput() {
        let value = formData.phone.replace(/\D/g, '');

        if (value.length > 3 && value.length <= 6) {
            value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
        } else if (value.length > 6) {
            value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6, 10)}`;
        } else if (value.length > 3) {
            value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
        }

        formData.phone = value;
        validatePhoneNumber();
    }

    function validatePhoneNumber(): boolean {
        const phoneRegex = /^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$/;

        if (formData.phone === '') {
            phoneError = '';
            
            return false;
        } else if (!phoneRegex.test(formData.phone)) {
            phoneError = 'Invalid phone number';
            return false;
        } else {
            phoneError = '';
            return true;
        }
    }

    // Logo upload
    function handleLogoUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        if (input.files && input.files[0]) {
            const reader = new FileReader();
            reader.onload = (e) => {
                formData.logo = e.target?.result as string;
            };
            reader.readAsDataURL(input.files[0]);
        }
    }

    function triggerFileUpload() {
        if (companyEditMode) {
            const fileInput = document.getElementById('fileInput') as HTMLInputElement;
            fileInput.click();
        }
    }

    // Edit mode toggles
    // function toggleCompanyEditMode() {
    //     if (companyEditMode) {
            
    //         passwordEditable = false;
    //         if (validateRequiredFields('company')) {
    //             showSuccessModal = true;
    //             companyEditMode = false;
    //             setTimeout(() => showSuccessModal = false, 2000);
    //         }
    //     } else {
    //         companyEditMode = true;
    //         customerEditMode = false;
    //         passwordEditable = false;
    //     }
    // }

    // function toggleCustomerEditMode() {
    //     if (customerEditMode) {
    //         if (validatePhoneNumber() && validateRequiredFields('customer')) {
    //             showSuccessModal = true;
    //             customerEditMode = false;
    //             setTimeout(() => showSuccessModal = false, 2000);
    //         }
    //     } else {
    //         customerEditMode = true;
    //         companyEditMode = false;
    //     }
    // }

    function toggleCompanyEditMode() {
    if (companyEditMode) {
      if (validateCompanyFields()) {
        callCompanyAPI();
        companyEditMode = false;
        passwordEditable = false;
      }
    } else {
      companyEditMode = true;
      customerEditMode = false;
    }
  }

  function toggleCustomerEditMode() {
    if (customerEditMode) {
      if (validateCustomerFields()) {
        callCustomerAPI();
        customerEditMode = false;
      }
    } else {
      customerEditMode = true;
      companyEditMode = false;
    }
  }

    // Validation
    function validateRequiredFields(section: 'company' | 'customer'): boolean {
        const companyFields = ['companyName', 'username', 'companyStreet', 'companyCity', 'companyState', 'companyZip'];
        const customerFields = ['firstName', 'lastName', 'email', 'phone', 'customerStreet', 'customerCity', 'customerState', 'customerZip'];
        const fieldsToCheck = section === 'company' ? companyFields : customerFields;

        return fieldsToCheck.every(field => !!formData[field as keyof typeof formData]);
    }

    // Switch sections
    function showSection(section: 'company' | 'customer') {
        activeSection = section;
        companyEditMode = false;
        customerEditMode = false;
        passwordEditable = false;
    }

    // Placeholder: implement this as needed
    function loadCachedData(company: any, customer: any, admin: any) {
        // Populate formData from cache
        // (You can customize this based on your structure)
    }

    async function fetchProfileData() {
        // Placeholder for actual profile data fetch logic
        isLoading = false;
    }
</script>


<div class="min-h-screen bg-gray-100">
    <!-- Loading Overlay -->
    {#if isLoading}
        <div class="fixed inset-0 z-50 flex items-center justify-center"
         style="background: rgba(0, 0, 0, 0.5)">
            <div class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"></div>
        </div>
    {/if}

    <!-- Success Modal -->
    {#if showSuccessModal}
        <div
            transition:fade
            class="fixed inset-0 z-50 flex items-center justify-center"
            style="background: rgba(0, 0, 0, 0.5)"
        >
            <div class="bg-white rounded-lg shadow-xl max-w-md w-full mx-4">
                <div class="bg-[#02066F] text-white p-4 rounded-t-lg">
                    <h5 class="text-lg font-bold text-center">Success!</h5>
                </div>
                <div class="p-10">
                    <p class="text-center font-semibold mb-4">Your details have been updated successfully!</p>
                </div>
            </div>
        </div>
    {/if}

    <!-- Main Content -->
    <div class="max-w-4xl w-full mx-auto px-4 py-8 pt-28">
        <!-- Section Toggle Buttons -->
        <div class="text-center mb-6 space-y-2">
            <button 
                class={`px-6 py-3 rounded-xl mr-2 cursor-pointer ${activeSection === 'company' ? 'bg-[#02066F] text-white' : 'bg-white text-[#02066F] border border-[#02066F]'}`}
                on:click={() => showSection('company')}
            >
                Company Details
            </button>
            <button 
                class={`px-6 py-3 rounded-xl cursor-pointer ${activeSection === 'customer' ? 'bg-[#02066F] text-white' : 'bg-white text-[#02066F] border border-[#02066F]'}`}
                on:click={() => showSection('customer')}
            >
                Customer Details
            </button>
        </div>

        <h1 class="text-3xl font-bold text-center text-gray-800 mb-6">Profile</h1>

        <!-- Company Section -->
        {#if activeSection === 'company'}
            <!-- Logo Section -->
            <div class="flex justify-center mb-8">
                <div class="relative w-24 h-24 rounded-full border-2 border-[#02066F] overflow-hidden">
                    {#if formData.logo}
                        <img src={formData.logo} alt="Company Logo" class="w-full h-full object-cover" />
                    {:else}
                        <div class="w-full h-full bg-gray-100 flex items-center justify-center">
                            <i class="fas fa-building text-3xl text-gray-500"></i>
                        </div>
                    {/if}
                    {#if companyEditMode}
                        <div 
                            class="absolute bottom-0 right-0 bg-white p-2 rounded-full cursor-pointer"
                            on:click={triggerFileUpload}
                        >
                            <i class="fas fa-pencil-alt text-[#02066F]"></i>
                        </div>
                        <input
                            id="fileInput"
                            type="file"
                            accept="image/*"
                            on:change={handleLogoUpload}
                            class="hidden"
                        />
                    {/if}
                </div>
            </div>

            <div class="bg-white rounded-xl shadow-lg overflow-hidden p-6 pb-10 mb-8">
                <!-- Company Form -->
                <form id="companyForm">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company Name:</label>
                            <input
                                bind:value={formData.companyName}
                                type="text"
                                placeholder="Company Name"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!companyEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!companyEditMode}
                                required
                                 on:blur={() => {
                                    if (companyEditMode && !formData.companyName.trim()) {
                                        errors.companyName = 'Please fill out this field';
                                    } else {
                                        errors.companyName = '';
                                    }
                                    }}
                            />
                             {#if errors.companyName}
                                <p class="text-red-500 text-sm mt-1">{errors.companyName}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company Street:</label>
                            <input
                                bind:value={formData.companyStreet}
                                type="text"
                                placeholder="Company Street"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!companyEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!companyEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.companyStreet.trim()) {
                                        errors.companyStreet = 'Please fill out this field';
                                    } else {
                                        errors.companyStreet = '';
                                    }
                                    }}
                                
                            />
                             {#if errors.companyStreet}
                                <p class="text-red-500 text-sm mt-1">{errors.companyStreet}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company State:</label>
                            <input
                                bind:value={formData.companyState}
                                type="text"
                                placeholder="Company State"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!companyEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!companyEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.companyState.trim()) {
                                        errors.companyState = 'Please fill out this field';
                                    } else {
                                        errors.companyState = '';
                                    }
                                    }}
                            />
                             {#if errors.companyState}
                                <p class="text-red-500 text-sm mt-1">{errors.companyState}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company City:</label>
                            <input
                                bind:value={formData.companyCity}
                                type="text"
                                placeholder="Company City"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!companyEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!companyEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.companyCity.trim()) {
                                        errors.companyCity = 'Please fill out this field';
                                    } else {
                                        errors.companyCity = '';
                                    }
                                    }}
                            />
                             {#if errors.companyCity}
                                <p class="text-red-500 text-sm mt-1">{errors.companyCity}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company Zip Code:</label>
                            <input
                                bind:value={formData.companyZip}
                                type="text"
                                placeholder="Company Zip"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!companyEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!companyEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.companyZip.trim()) {
                                        errors.companyZip = 'Please fill out this field';
                                    } else {
                                        errors.companyZip = '';
                                    }
                                    }}
                            />
                             {#if errors.companyZip}
                                <p class="text-red-500 text-sm mt-1">{errors.companyZip}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Username:</label>
                            <input
                                bind:value={formData.username}
                                type="text"
                                placeholder="Username"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!companyEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!companyEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.username.trim()) {
                                        errors.username = 'Please fill out this field';
                                    } else {
                                        errors.username = '';
                                    }
                                    }}
                            />
                             {#if errors.username}
                                <p class="text-red-500 text-sm mt-1">{errors.username}</p>
                            {/if}
                        </div>
                        <!-- <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Password:</label>
                            <input
                                bind:value={formData.password}
                                type="password"
                                placeholder="Password"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!companyEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!companyEditMode}
                                required
                            />
                        </div> -->
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Password:</label>
                            <input
                                bind:value={formData.password}
                                type="password"
                                id="passwordField"
                                placeholder="Password"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!passwordEditable ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!passwordEditable}
                                readonly={!passwordEditable}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.password.trim()) {
                                        errors.password = 'Please fill out this field';
                                    } else {
                                        errors.password = '';
                                    }
                                    }}
                            />
                             {#if errors.password}
                                <p class="text-red-500 text-sm mt-1">{errors.password}</p>
                            {/if}
                        </div>
                        <div class="flex items-center text-center justify-center">
                            <!-- {#if companyEditMode} -->
                                <button
                                    type="button"
                                    class="px-6 py-3 bg-[#02066F] text-white rounded-xl cursor-pointer"
                                    disabled={!companyEditMode}
                                    on:click={() => {
                                        passwordEditable = !passwordEditable;
                                        // const passwordInput = document.getElementById('password') as HTMLInputElement | null;
                                        // if (passwordInput) {
                                        //     passwordInput.disabled = !passwordInput.disabled;
                                        // }
                                        if (passwordEditable) {
                                        setTimeout(() => {
                                            const passwordInput = document.getElementById('passwordField');
                                            if (passwordInput) passwordInput.focus();
                                        }, 0);
                                    }
                                    }}
                                >
                                
                                    Change Password
                                </button>
                            <!-- {/if} -->
                        </div>
                    </div>
                </form>
            </div>
            
            <!-- Form Actions -->
            <div class="flex justify-center mt-10 pb-16">
                <button
                    on:click={toggleCompanyEditMode}
                    class="w-full px-8 py-3 bg-[#02066F] cursor-pointer text-white rounded-lg text-lg font-bold"
                >
                    {companyEditMode ? 'Save' : 'Edit'}
                </button>
            </div>
            
        {:else}
            <!-- Customer Section -->
            <div class="bg-white rounded-xl shadow-lg overflow-hidden p-6 mb-8">
                <form id="customerForm">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">First Name:</label>
                            <input
                                bind:value={formData.firstName}
                                type="text"
                                placeholder="First Name"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={() => {
                                if (companyEditMode && !formData.firstName.trim()) {
                                    errors.firstName = 'Please fill out this field';
                                } else {
                                    errors.firstName = '';
                                }
                                }}
                            />
                             {#if errors.firstName}
                                <p class="text-red-500 text-sm mt-1">{errors.firstName}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Last Name:</label>
                            <input
                                bind:value={formData.lastName}
                                type="text"
                                placeholder="Last Name"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.lastName.trim()) {
                                        errors.lastName = 'Please fill out this field';
                                    } else {
                                        errors.lastName = '';
                                    }
                                    }}
                            />
                             {#if errors.lastName}
                                <p class="text-red-500 text-sm mt-1">{errors.lastName}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Email:</label>
                            <input
                                bind:value={formData.email}
                                type="email"
                                placeholder="Email"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.email.trim()) {
                                        errors.email = 'Please fill out this field';
                                    } else {
                                        errors.email = '';
                                    }
                                    }}
                            />
                             {#if errors.email}
                                <p class="text-red-500 text-sm mt-1">{errors.email}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Phone Number:</label>
                            <input
                                bind:value={formData.phone}
                                on:input={formatPhoneNumberInput}
                                type="tel"
                                placeholder="Phone Number"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={validatePhoneNumber}
                            />
                             {#if errors.phone}
                                <p class="text-red-500 text-sm mt-1">{errors.phone}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer Street:</label>
                            <input
                                bind:value={formData.customerStreet}
                                type="text"
                                placeholder="Customer Street"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.customerStreet.trim()) {
                                        errors.customerStreet = 'Please fill out this field';
                                    } else {
                                        errors.customerStreet = '';
                                    }
                                    }}
                            />
                             {#if errors.customerStreet}
                                <p class="text-red-500 text-sm mt-1">{errors.customerStreet}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer State:</label>
                            <input
                                bind:value={formData.customerState}
                                type="text"
                                placeholder="Customer State"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.customerState.trim()) {
                                        errors.customerState = 'Please fill out this field';
                                    } else {
                                        errors.customerState = '';
                                    }
                                    }}
                            />
                             {#if errors.customerState}
                                <p class="text-red-500 text-sm mt-1">{errors.customerState}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer City:</label>
                            <input
                                bind:value={formData.customerCity}
                                type="text"
                                placeholder="Customer City"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.customerCity.trim()) {
                                        errors.customerCity = 'Please fill out this field';
                                    } else {
                                        errors.customerCity = '';
                                    }
                                    }}
                            />
                             {#if errors.customerCity}
                                <p class="text-red-500 text-sm mt-1">{errors.customerCity}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer Zip Code:</label>
                            <input
                                bind:value={formData.customerZip}
                                type="text"
                                placeholder="Customer Zip"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={() => {
                                    if (companyEditMode && !formData.customerZip.trim()) {
                                        errors.customerZip = 'Please fill out this field';
                                    } else {
                                        errors.customerZip = '';
                                    }
                                    }}
                            />
                             {#if errors.customerZip}
                                <p class="text-red-500 text-sm mt-1">{errors.customerZip}</p>
                            {/if}
                        </div>
                    </div>
                </form>
            </div>
            
            <!-- Form Actions -->
            <div class="flex justify-center mt-10 pb-16">
                <button
                    on:click={toggleCustomerEditMode}
                    class="w-full px-8 py-3 bg-[#02066F] cursor-pointer text-white rounded-lg text-lg font-bold"
                >
                    {customerEditMode ? 'Save' : 'Edit'}
                </button>
            </div>
        {/if}
    </div>
</div>