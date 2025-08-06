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
    let userType: string = '';
    let passwordEditable = false;
    let showPassword = false;

    // API URLs
    const customerAPIUrlBase = `https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/customer`;
    const companyAPIUrlBase = `https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/company`;

    // Form data structure
    let formData = {
        companyName: '',
        companyStreet: '',
        companyCity: '',
        companyState: '',
        companyZip: '',
        firstName: '',
        lastName: '',
        EName: '',
        customerStreet: '',
        customerCity: '',
        customerState: '',
        customerZip: '',
        email: '',
        phone: '',
        logo: '',
        adminPin: '',
        decryptedPassword: '', // Added property
    };

let errors = {

    companyName: '',
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
    adminPin: '',
    EName: ''
  };



    // Local storage data
    let companyId = '';
    let customerId = '';
    let adminEmail = '';

    
    onMount(async () => {
    companyId = localStorage.getItem('companyID') || '';
    customerId = localStorage.getItem('customerID') || '';
    adminEmail = localStorage.getItem('adminMail') || '';
    userType = localStorage.getItem('adminType') || '';

    // First load from localStorage
    const companyData = localStorage.getItem("companyProfileData");
    const customerData = localStorage.getItem("customerProfileData");
    const storedAdmin = localStorage.getItem('loggedAdmin');
const adminDetails = storedAdmin ? JSON.parse(storedAdmin) : null;
    const companyAddress = localStorage.getItem('companyAddress') || '';
    const customerAddress = localStorage.getItem('address') || '';
    const [companyStreet, companyCity, companyState, companyZip] = companyAddress.split('--');
    const [customerStreet, customerCity, customerState, customerZip] = customerAddress.split('--');

    formData = {
    ...formData,
    companyName: localStorage.getItem('companyName') || '',
    companyStreet: companyStreet || '',
    companyCity: companyCity || '',
    companyState: companyState || '',
    companyZip: companyZip || '',
    logo: localStorage.getItem('companyLogo') || '',
};

// console.log('Company Data:', adminDetails);

  if(adminDetails){
    formData = {
   ...formData,
EName: `${adminDetails.FName || ''} ${adminDetails.LName || ''}`.trim(),
adminPin: adminDetails.Pin || '',
email: adminDetails.Email || '',
phone: adminDetails.PhoneNumber || ''

    };
  }
  //Customer Data
  else{
    formData = {
    ...formData,
    firstName: localStorage.getItem('firstName') || '',
    lastName: localStorage.getItem('lastName') || '',
    customerStreet: customerStreet || '',
    customerCity: customerCity || '',
    customerState: customerState || '',
    customerZip: customerZip || '',
    email: localStorage.getItem('email') || '',
    phone: localStorage.getItem('phone') || ''
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
    // async function encryptPasswordInput(): Promise<string> {
    //     const password = formData.decryptedPassword;
    //     const storedPassword = localStorage.getItem('passwordDecryptedValue');

    //     if (password === storedPassword) {
    //         return localStorage.getItem('password') || '';
    //     }

    //     const keyValue = new Uint8Array([16, 147, 220, 113, 166, 142, 22, 93, 241, 91, 13, 252, 112, 122, 119, 95]);
    //     return await encryptData(password, keyValue);
    // }

    // async function encryptData(data: string, key: Uint8Array): Promise<string> {
    //     const dataBuffer = new TextEncoder().encode(data);
    //     const algorithm = { name: 'AES-GCM', iv: generateRandomBytes(12) };

    //     // Ensure key is a plain Uint8Array (not a generic or typed variant)
    //     const importedKey = await crypto.subtle.importKey(
    //         'raw', key.buffer, { name: 'AES-GCM' }, false, ['encrypt']
    //     );

    //     const encryptedData = await crypto.subtle.encrypt(
    //         algorithm, importedKey, dataBuffer
    //     );

    //     const iv = algorithm.iv;
    //     const encryptedDataWithIV = new Uint8Array(iv.byteLength + encryptedData.byteLength);
    //     encryptedDataWithIV.set(iv);
    //     encryptedDataWithIV.set(new Uint8Array(encryptedData), iv.byteLength);

    //     return btoa(String.fromCharCode(...new Uint8Array(encryptedDataWithIV)));
    // }

    // function generateRandomBytes(length: number): Uint8Array {
    //     const randomValues = new Uint8Array(length);
    //     crypto.getRandomValues(randomValues);
    //     return randomValues;
    // }

    // API functions
    async function callCompanyAPI() {
        if (!companyId) return;
        // const encryptedPassword = await encryptPasswordInput();
        const passwordEncryptVal = localStorage.getItem('password');
        const unameLocalStorage = localStorage.getItem('username');
        const companyData = {
            CID: companyId,
            UserName: unameLocalStorage,
            CName: formData.companyName,
            CAddress: `${formData.companyStreet}--${formData.companyCity}--${formData.companyState}--${formData.companyZip}`,
            CLogo: formData.logo || localStorage.getItem("companyLogo"),
            Password: passwordEncryptVal,
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
            console.log("Company data updated successfully:", companyData);
             showSuccessModal = true;
            setTimeout(() => showSuccessModal = false, 2000);
            
        } catch (error) {
            console.error("Company API Error:", error);
        }
    }

    console.log("Company Logo:",formData.logo === "");
    

    async function callCustomerAPI() {
        if (!companyId || !customerId) return;
        const customerData = {
            CustomerID: customerId,
            CID: companyId,
            FName: formData.firstName,
            LName: formData.lastName,
            Address: `${formData.customerStreet}--${formData.customerCity}--${formData.customerState}--${formData.customerZip}`,
            PhoneNumber: formData.phone,
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
     let fileInput: HTMLInputElement;

    function handleLogoUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        if (input.files && input.files[0]) {
            const reader = new FileReader();
            reader.onload = (e) => {
                formData.logo = e.target?.result as string;
                localStorage.setItem('companyLogo', formData.logo); // Save to localStorage
            };
            reader.readAsDataURL(input.files[0]);
        }
    }

    function triggerFileUpload() {
        if (companyEditMode && fileInput) {
            fileInput.click();
        }
    }


    function toggleCompanyEditMode() {
    if (userType == 'Owner' && companyEditMode) {
      if (validateCompanyFields()) {
        callCompanyAPI();
        companyEditMode = false;
        passwordEditable = false;
        showSuccessModal = true;
        setTimeout(() => showSuccessModal = false, 2000);
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

        showSuccessModal = true;

        showSuccessModal= true;

        setTimeout(() => showSuccessModal = false, 2000);
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

    console.log("logo:",formData.logo === '');
    
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
            {userType === 'Owner' ? 'Customer Details' : (userType === 'Admin' ? 'Admin Details' : 'Super Admin Details')}
            </button>
        </div>

        <h1 class="text-3xl font-bold text-center text-gray-800 mb-6">Profile</h1>

        <!-- Company Section -->
        {#if activeSection === 'company'}
            <!-- Logo Section -->
            <div class="flex justify-center mb-8">
                <div 
                    class="relative w-24 h-24 rounded-full border-2 border-[#02066F] overflow-hidden 
                        {companyEditMode ? 'cursor-pointer bg-white' : 'bg-gray-200 cursor-not-allowed'}"
                    on:click={() => companyEditMode && triggerFileUpload()}
                >
                    {#if typeof(formData.logo) === 'string'}
                        <div class="w-full h-full bg-gray-100 flex items-center justify-center">
                            <i class="fas fa-building text-3xl text-gray-500"></i>
                        </div>
                    {:else}
                        <img src={formData.logo} alt="Company Logo" class="w-full h-full object-cover" />
                    {/if}

                    <input
                        bind:this={fileInput}
                        type="file"
                        accept="image/*"
                        on:change={handleLogoUpload}
                        class="hidden"
                    />
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
                            <label class="block text-base font-bold text-gray-900 mb-1">Company Address Line 1:</label>
                            <input
                                bind:value={formData.companyStreet}
                                type="text"
                                placeholder="Company Address Line 1"
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
            {#if userType !== 'Admin' && userType !== 'SuperAdmin'}
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
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer Address Line 1:</label>
                            <input
                                bind:value={formData.customerStreet}
                                type="text"
                                placeholder="Customer Address Line 1"
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
                {:else}
                <form id="adminForm">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <!-- Pin -->
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Pin:</label>
                            <input
                                bind:value={formData.adminPin}
                                type="number"
                                placeholder="Pin"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
                                disabled={!customerEditMode}
                                required
                                on:blur={() => {
                                    if (customerEditMode && !formData.adminPin) {
                                        errors.adminPin = 'Please fill out this field';
                                    } else {
                                        errors.adminPin = '';
                                    }
                                }}
                            />
                            {#if errors.adminPin}
                                <p class="text-red-500 text-sm mt-1">{errors.adminPin}</p>
                            {/if}
                        </div>

                        <!-- Last Name -->
                      <div>
    <label class="block text-base font-bold text-gray-900 mb-1">Employee Name:</label>
    <input
        bind:value={formData.EName}
        type="text"
        placeholder="Employee Name"
        class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!customerEditMode ? 'bg-gray-200 text-gray-500' : ''}`}
        disabled={!customerEditMode}
        required
        on:blur={() => {
            if (customerEditMode && !formData.EName.trim()) {
                errors.EName = 'Please fill out this field';
            } else {
                errors.EName = '';
            }
        }}
    />
    {#if errors.EName}
        <p class="text-red-500 text-sm mt-1">{errors.EName}</p>
    {/if}
</div>


                        <!-- Email -->
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
                                    if (customerEditMode && !formData.email.trim()) {
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

                        <!-- Phone -->
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
                                on:blur={() => {
                                    if (customerEditMode && !formData.phone.trim()) {
                                        errors.phone = 'Please fill out this field';
                                    } else {
                                        errors.phone = '';
                                    }
                                }}
                            />
                            {#if errors.phone}
                                <p class="text-red-500 text-sm mt-1">{errors.phone}</p>
                            {/if}
                        </div>
                    </div>
                </form>
                {/if}
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