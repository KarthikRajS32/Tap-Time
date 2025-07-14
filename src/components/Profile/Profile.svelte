<script lang="ts">
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';
    
    // State variables
    let isLoading = true;
    let editMode = false;
    let showSuccessModal = false;
    let phoneError = '';
    let activeSection: 'company' | 'customer' = 'company';
    
    // Form data structure
    let formData = {
        companyName: '',
        username: '',
        password: '',
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
        logo: ''
    };

    // Load profile data on component mount
    onMount(async () => {
        try {
            // Simulate API loading
            await new Promise(resolve => setTimeout(resolve, 1000));
            isLoading = false;
            
            // Mock data - replace with actual API calls
            formData = {
                companyName: 'Arjava Technologies Pvt Ltd',
                username: 'testusersecond',
                password: 'password123',
                companyStreet: '215th',
                companyCity: 'fdvtfdv',
                companyState: 'bgrr',
                companyZip: 'hhh',
                firstName: 'Arjava',
                lastName: 'Technologies',
                customerStreet: 'Arjava Tech India (Pvt) Limited',
                customerCity: 'Selaiyur',
                customerState: 'TamilNadu',
                customerZip: '600073',
                email: 'mani@arjavatech.com',
                phone: '',
                logo: ''
            };
        } catch (error) {
            console.error('Error loading profile data:', error);
            isLoading = false;
        }
    });

    // Format phone number for display
    function formatPhoneNumberForDisplay(phoneNumber: string): string {
        const cleaned = phoneNumber.replace(/\D/g, '');
        if (cleaned.length !== 10) return phoneNumber;
        
        const areaCode = cleaned.substring(0, 3);
        const centralOfficeCode = cleaned.substring(3, 6);
        const lineNumber = cleaned.substring(6);
        
        return `(${areaCode}) ${centralOfficeCode}-${lineNumber}`;
    }

    // Handle phone number input formatting
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

    // Validate phone number format
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

    // Handle logo upload
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

    // Trigger file input click
    function triggerFileUpload() {
        if (editMode) {
            const fileInput = document.getElementById('fileInput') as HTMLInputElement;
            fileInput.click();
        }
    }

    // Toggle edit/save mode
    function toggleEditMode() {
        if (editMode) {
            if (validatePhoneNumber() && validateRequiredFields()) {
                showSuccessModal = true;
                editMode = false;
                setTimeout(() => showSuccessModal = false, 2000);
            }
        } else {
            editMode = true;
        }
    }

    // Validate all required fields
    function validateRequiredFields(): boolean {
        const requiredFields = [
            'companyName', 'username', 'companyStreet', 'companyCity',
            'companyState', 'companyZip', 'firstName', 'lastName',
            'email', 'phone'
        ];
        
        return requiredFields.every(field => {
            if (!formData[field as keyof typeof formData]) {
                return false;
            }
            return true;
        });
    }

    // Switch between company and customer sections
    function showSection(section: 'company' | 'customer') {
        activeSection = section;
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
                    <!-- <button 
                        type="button" 
                        class="absolute top-3 right-3 text-white"
                        on:click={() => showSuccessModal = false}
                    >
                        <i class="fas fa-times"></i>
                    </button> -->
                </div>
                <div class="p-10">
                    <p class="text-center font-semibold mb-4">Your details have been updated successfully!</p>
                    <!-- <div class="flex justify-center">
                        <button 
                            type="button" 
                            class="px-6 py-2 bg-gray-300 rounded-lg"
                            on:click={() => showSuccessModal = false}
                        >
                            Close
                        </button>
                    </div> -->
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
                        {#if editMode}
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
                <form id="companyForm" on:submit|preventDefault={toggleEditMode}>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company Name:</label>
                            <input
                                bind:value={formData.companyName}
                                type="text"
                                placeholder="Company Name"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company Street:</label>
                            <input
                                bind:value={formData.companyStreet}
                                type="text"
                                placeholder="Company Street"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company State:</label>
                            <input
                                bind:value={formData.companyState}
                                type="text"
                                placeholder="Company State"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company City:</label>
                            <input
                                bind:value={formData.companyCity}
                                type="text"
                                placeholder="Company City"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Company Zip Code:</label>
                            <input
                                bind:value={formData.companyZip}
                                type="text"
                                placeholder="Company Zip"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Username:</label>
                            <input
                                bind:value={formData.username}
                                type="text"
                                placeholder="Username"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Password:</label>
                            <input
                                bind:value={formData.password}
                                type="password"
                                placeholder="Password"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div class="flex items-center text-center justify-center">
                            <!-- {#if editMode} -->
                                <button
                                    type="button"
                                    class="px-6 py-3 bg-[#02066F] text-white rounded-xl cursor-pointer"
                                    on:click={() => {
                                        const passwordInput = document.getElementById('password') as HTMLInputElement | null;
                                        if (passwordInput) {
                                            passwordInput.disabled = !passwordInput.disabled;
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
                            form="companyForm"
                            type="submit"
                            class="w-full px-8 py-3 bg-[#02066F] cursor-pointer text-white rounded-lg text-lg font-bold"
                        >
                            {editMode ? 'Save' : 'Edit'}
                        </button>
                    </div>
            
        {:else}
            <!-- Customer Section -->
            <div class="bg-white rounded-xl shadow-lg overflow-hidden p-6 mb-8">
                <form id="customerForm" on:submit|preventDefault={toggleEditMode}>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">First Name:</label>
                            <input
                                bind:value={formData.firstName}
                                type="text"
                                placeholder="First Name"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Last Name:</label>
                            <input
                                bind:value={formData.lastName}
                                type="text"
                                placeholder="Last Name"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Email:</label>
                            <input
                                bind:value={formData.email}
                                type="email"
                                placeholder="Email"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Phone Number:</label>
                            <input
                                bind:value={formData.phone}
                                on:input={formatPhoneNumberInput}
                                type="tel"
                                placeholder="Phone Number"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                            {#if phoneError}
                                <p class="text-red-500 text-sm mt-1">{phoneError}</p>
                            {/if}
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer Street:</label>
                            <input
                                bind:value={formData.customerStreet}
                                type="text"
                                placeholder="Customer Street"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer State:</label>
                            <input
                                bind:value={formData.customerState}
                                type="text"
                                placeholder="Customer State"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer City:</label>
                            <input
                                bind:value={formData.customerCity}
                                type="text"
                                placeholder="Customer City"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                        <div>
                            <label class="block text-base font-bold text-gray-900 mb-1">Customer Zip Code:</label>
                            <input
                                bind:value={formData.customerZip}
                                type="text"
                                placeholder="Customer Zip"
                                class={`w-full px-4 py-3 rounded-lg border border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 focus:ring-black ${!editMode ? 'bg-gray-50' : ''}`}
                                disabled={!editMode}
                                required
                            />
                        </div>
                    </div>

                   
                </form>
            </div>
             <!-- Form Actions -->
                    <div class="flex justify-center mt-10 pb-16">
                        <button
                            form="customerForm"
                            type="submit"
                            class="w-full px-8 py-3 bg-[#02066F] cursor-pointer text-white rounded-lg text-lg font-bold"
                        >
                            {editMode ? 'Save' : 'Edit'}
                        </button>
                    </div>
        {/if}
    </div>
</div>