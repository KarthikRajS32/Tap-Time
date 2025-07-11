<script lang="ts">
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';
  
    // State variables
    let isLoading = true;
    let editMode = false;
    let showSuccessModal = false;
    let phoneError = '';
    
    // Form data structure
    let formData = {
      companyName: 'Arjava Technologies Pvt Ltd',
      username: 'testusersecond',
      companyStreet: '215th',
      companyCity: '',
      companyState: '',
      companyZip: '',
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
  
    // Load profile data on component mount
    onMount(async () => {
      try {
        // Simulate API loading
        await new Promise(resolve => setTimeout(resolve, 1000));
        isLoading = false;
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
  </script>


  
  <div class="min-h-screen bg-gray-100">
  
    <!-- Success Modal -->
    {#if showSuccessModal}
      <div
        transition:fade
        class="fixed inset-0 z-50 flex items-center justify-center "
        style="background: rgba(0, 0, 0, 0.5)"
      >
        <div class="bg-white rounded-lg shadow-xl max-w-md w-full mx-4">
          <div class="p-6">
            <h3 class="text-lg font-bold text-gray-900 mb-4">Success!</h3>
            <p class="text-gray-700">Your profile has been updated successfully.</p>
          </div>
        </div>
      </div>
    {/if}
  
    <!-- Main Content -->
    <div class="max-w-5xl w-full mx-auto px-10 py-8 pt-24">
      <h1 class="text-3xl font-bold text-center text-gray-800 mb-4">Profile</h1>
  
      <!-- Logo Section -->
      <div class="flex justify-center mb-8">
        <div class=" rounded-full border-2 border-[#02066F] overflow-hidden">
          {#if formData.logo}
            <img src={formData.logo} alt="Company Logo" class="w-30 h-30 object-cover rounded-full" />
          {:else}
            <div class="w-full h-full p-6 bg-gray-300 flex items-center justify-center rounded-full">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-12 w-12 text-gray-500"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
                />
              </svg>
            </div>
          {/if}
                    {#if editMode}
            <div class="relative">
              <!-- Assuming your logo is in a container with class "logo-container" -->
              <button
                on:click={triggerFileUpload}
                class="absolute -bottom-1 left-2/3 transform -translate-x-1/2 bg-white px-2 py-1 rounded-full text-[#02066F] hover:bg-gray-100 border border-gray-200 shadow-sm"
              >
                <i class="fas fa-pencil-alt text-base cursor-pointer"></i>
              </button>
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
  
      <!-- Profile Form -->
      <div class="max-w-2xl mx-auto bg-white rounded-xl shadow-lg overflow-hidden p-2">
        <form on:submit|preventDefault={toggleEditMode} class="">
          <!-- Company Information Section -->
          <div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 p-6 md:p-14 space-y-2">
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Company Name:</label>
                <input
                  bind:value={formData.companyName}
                  type="text"
                  placeholder="Company Name"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1  ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Username:</label>
                <input
                  bind:value={formData.username}
                  type="text"
                  placeholder="Username"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1  ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Company Street:</label>
                <input
                  bind:value={formData.companyStreet}
                  type="text"
                  placeholder="Company Street"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1  ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Company City:</label>
                <input
                  bind:value={formData.companyCity}
                  type="text"
                  placeholder="Company City"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1  ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Company State:</label>
                <input
                  bind:value={formData.companyState}
                  type="text"
                  placeholder="Company State"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1  ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Company Zip Code:</label>
                <input
                  bind:value={formData.companyZip}
                  type="text"
                  placeholder="Company Zip"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Email:</label>
                <input
                  bind:value={formData.email}
                  type="email"
                  placeholder="Email"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Phone Number:</label>
                <input
                  bind:value={formData.phone}
                  on:input={formatPhoneNumberInput}
                  type="tel"
                  placeholder="Phone Number"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
                {#if phoneError}
                  <p class="mt-1 text-sm text-red-600">{phoneError}</p>
                {/if}
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">First Name:</label>
                <input
                  bind:value={formData.firstName}
                  type="text"
                  placeholder="First Name"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Last Name:</label>
                <input
                  bind:value={formData.lastName}
                  type="text"
                  placeholder="Last Name"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Customer Street:</label>
                <input
                  bind:value={formData.customerStreet}
                  type="text"
                  placeholder="Customer Street"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Customer City:</label>
                <input
                  bind:value={formData.customerCity}
                  type="text"
                  placeholder="Customer City"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Customer State:</label>
                <input
                  bind:value={formData.customerState}
                  type="text"
                  placeholder="Customer State"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
              <div>
                <label class="block text-md font-bold text-gray-800 mb-1">Customer Zip Code:</label>
                <input
                  bind:value={formData.customerZip}
                  type="text"
                  placeholder="Customer Zip"
                  class={`w-full px-4 py-3 rounded-lg border-1 border-[#02066F] text-[#02066F] font-bold focus:outline-none focus:ring-1 ${!editMode ? 'bg-gray-50' : ''}`}
                  disabled={!editMode}
                  required
                />
              </div>
            </div>
          </div>

  
          <!-- Form Actions -->
          <div class="flex justify-center space-x-4 pb-4">

            <button
              type="submit"
              class="px-6 py-2 bg-[#02066F] text-white rounded-lg transition-colors cursor-pointer"
            >
              {editMode ? 'Save' : 'Edit'}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>