<script lang="ts">
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';
  
    type Employee = {
      pin: string;
      name: string;
      phone: string;
      isAdmin: boolean;
    };
  
    // Sample data
    let employees: Employee[] = [
    //   { pin: '1234', name: 'John Doe', phone: '(123) 456-7890', isAdmin: false },
    //   { pin: '5678', name: 'Jane Smith', phone: '(234) 567-8901', isAdmin: false },
    //   { pin: '9012', name: 'Admin User', phone: '(345) 678-9012', isAdmin: true }
    ];
  
    let filteredEmployees: Employee[] = [];
    let admins: Employee[] = [];
    let searchTerm: string = '';
    let showEmployeeModal = false;
    let showAdminModal = false;
    let showDeleteModal = false;
    let currentEmployee: Employee | null = null;
    let isEditing = false;
    let loading = true;
    let adminCount = 0;
  
    // Form fields
    let formData = {
      pin: '',
      firstName: '',
      lastName: '',
      phone: '',
      isAdmin: false
    };
  
    // Validation errors
    let errors = {
      firstName: '',
      lastName: '',
      phone: ''
    };
  
    onMount(() => {
      setTimeout(() => {
        filterEmployees();
        loading = false;
      }, 1000);
    });
  
    function filterEmployees() {
      filteredEmployees = employees.filter(emp => !emp.isAdmin);
      admins = employees.filter(emp => emp.isAdmin);
      adminCount = admins.length;
    }
  
    function searchAdmins() {
      if (!searchTerm) {
        filterEmployees();
        return;
      }
      admins = employees.filter(emp =>
        emp.isAdmin &&
        (emp.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
          emp.pin.includes(searchTerm) ||
          emp.phone.includes(searchTerm))
      );
    }
  
    function formatPhoneNumber(phone: string): string {
      if (!phone) return '';
      let value = phone.replace(/\D/g, '');
      if (value.length > 6) {
        value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6, 10)}`;
      } else if (value.length > 3) {
        value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
      } else {
        value = `(${value}`;
      }
      return value;
    }
  
    function handlePhoneInput(e: Event) {
      const target = e.target as HTMLInputElement;
      let value = target.value.replace(/\D/g, '');
      if (value.length > 10) value = value.slice(0, 10);
  
      if (value.length > 6) {
        value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6)}`;
      } else if (value.length > 3) {
        value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
      } else if (value.length > 0) {
        value = `(${value}`;
      }
  
      formData.phone = value;
  
      if (value.length >= 13) {
        formData.pin = value.slice(-4);
      }
    }
  
    function validateForm(): boolean {
      let isValid = true;
      errors = { firstName: '', lastName: '', phone: '' };
  
      if (!formData.firstName.trim()) {
        errors.firstName = 'First name is required';
        isValid = false;
      } else if (!/^[a-zA-Z\s]+$/.test(formData.firstName)) {
        errors.firstName = 'Only letters allowed';
        isValid = false;
      }
  
      if (!formData.lastName.trim()) {
        errors.lastName = 'Last name is required';
        isValid = false;
      } else if (!/^[a-zA-Z\s]+$/.test(formData.lastName)) {
        errors.lastName = 'Only letters allowed';
        isValid = false;
      }
  
      const phoneRegex = /^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$/;
      if (!formData.phone) {
        errors.phone = 'Phone number is required';
        isValid = false;
      } else if (!phoneRegex.test(formData.phone)) {
        errors.phone = 'Invalid phone number format';
        isValid = false;
      }
  
      return isValid;
    }
  
    function openAddEmployee() {
      formData = {
        pin: '',
        firstName: '',
        lastName: '',
        phone: '',
        isAdmin: false
      };
      isEditing = false;
      showEmployeeModal = true;
    }
  
    function openAddAdmin() {
      formData = {
        pin: '',
        firstName: '',
        lastName: '',
        phone: '',
        isAdmin: true
      };
      isEditing = false;
      showAdminModal = true;
    }
  
    function openEditEmployee(employee: Employee) {
      currentEmployee = employee;
      const [firstName, ...rest] = employee.name.split(' ');
      formData = {
        pin: employee.pin,
        firstName: firstName,
        lastName: rest.join(' '),
        phone: employee.phone,
        isAdmin: employee.isAdmin
      };
      isEditing = true;
      if (employee.isAdmin) {
        showAdminModal = true;
      } else {
        showEmployeeModal = true;
      }
    }
  
    function openDeleteModal(employee: Employee) {
      currentEmployee = employee;
      showDeleteModal = true;
    }
  
    function submitForm() {
      if (!validateForm()) return;
  
      const fullName = `${formData.firstName} ${formData.lastName}`.trim();
      const newEmployee: Employee = {
        pin: formData.pin,
        name: fullName,
        phone: formData.phone,
        isAdmin: formData.isAdmin
      };
  
      if (isEditing && currentEmployee) {
        employees = employees.map(emp =>
          emp.pin === currentEmployee!.pin ? newEmployee : emp
        );
      } else {
        employees.push(newEmployee);
      }
  
      filterEmployees();
      showEmployeeModal = false;
      showAdminModal = false;
    }
  
    function deleteEmployee() {
      if (!currentEmployee) return;
      employees = employees.filter(emp => emp.pin !== currentEmployee!.pin);
      filterEmployees();
      showDeleteModal = false;
    }
  </script>
  

  <div class="min-h-screen h-full bg-gray-100 pt-28">

    <!-- Loading Overlay -->
    <!-- {#if loading}
      <div class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50">
        <div class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"></div>
      </div>
    {/if} -->
  
    <!-- Employee Section -->
    <div class="max-w-5xl mx-auto px-4 sm:px-6 md:px-8 mb-4">
        <div class="flex flex-row sm:flex-row items-center sm:justify-between justify-between gap-4 sm:gap-6 md:gap-8 text-center sm:text-left">
          <h2 class="text-xl sm:text-2xl md:text-3xl font-bold text-gray-800">
            Employee Details
          </h2>
          <button
            on:click={openAddEmployee}
            class="w-auto sm:w-auto px-6 py-2 text-sm sm:text-base bg-white border border-[#02066F] text-[#02066F] rounded-md transition-all duration-200 cursor-pointer"
          >
            Add Entry
          </button>
        </div>
    </div>


    <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-md overflow-hidden mb-8 border border-gray-400">
        <div class="p-4 sm:p-6 overflow-x-auto">
          <table class="w-full border-separate border-spacing-0 divide-y divide-gray-200">
                      
            <thead class="bg-[#02066F] text-white text-sm sm:text-base">
              <tr>
                <th class="px-4 sm:px-6 py-3 text-left font-semibold tracking-wide">Pin</th>
                <th class="px-4 sm:px-6 py-3 text-left font-semibold tracking-wide">Name</th>
                <th class="px-4 sm:px-6 py-3 text-left font-semibold tracking-wide">Phone Number</th>
                <th class="px-4 sm:px-6 py-3 text-left font-semibold tracking-wide">Action</th>
              </tr>
            </thead>
      
            <tbody class="bg-white divide-y divide-gray-200 text-sm">
              {#each filteredEmployees as employee (employee.pin)}
                <tr class="hover:bg-gray-100">
                  <td class="px-4 sm:px-6 py-4 whitespace-nowrap text-gray-900 text-center">{employee.pin}</td>
                  <td class="px-4 sm:px-6 py-4 whitespace-nowrap text-gray-900">{employee.name}</td>
                  <td class="px-4 sm:px-6 py-4 whitespace-nowrap text-gray-900">{employee.phone}</td>
                  <td class="px-4 sm:px-6 py-4 whitespace-nowrap text-gray-900">
                    <div class="flex space-x-6 justify-center text-center">
                      <button on:click={() => openEditEmployee(employee)} class="text-[#02066F] hover:text-[#02066F]">
                        <i class="fas fa-pencil-alt cursor-pointer"></i>
                      </button>
                      <button on:click={() => openDeleteModal(employee)} class="text-[#02066F] hover:text-[#02066F]">
                        <i class="fas fa-trash cursor-pointer"></i>
                      </button>
                    </div>
                  </td>
                </tr>
              {:else}
                <tr>
                  <td colspan="4" class="px-4 sm:px-6 py-4 text-center text-gray-500">No employees found</td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      </div>

      
    <!-- Admin Section -->
    <div class="max-w-5xl mx-auto px-4 sm:px-6 md:px-8 mb-4">
        <div class="flex flex-row sm:flex-row items-center sm:justify-between justify-between gap-4 sm:gap-6 md:gap-8 text-center sm:text-left">
        <h2 class="text-center md:text-left text-xl sm:text-2xl font-bold text-gray-800">
          Admin Details
        </h2>
          <button
            on:click={openAddAdmin}
            disabled={adminCount >= 3}
            class="w-auto sm:w-auto px-6 py-2 text-sm sm:text-base bg-white border border-[#02066F] text-[#02066F] rounded-md transition-all duration-200 cursor-pointer"
          >
            Add Entry
          </button>
        </div>
      </div>
      
     <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-md overflow-hidden mb-8 border border-gray-400">
        <div class="p-4 sm:p-6 overflow-x-auto">
          
          <!-- Search Input Responsive -->
          <div class="pb-4 flex flex-col sm:flex-row sm:items-center sm:justify-end gap-2">
            <label for="search" class="text-sm font-medium text-gray-700">Search:</label>
            <input
              id="search"
              type="text"
              bind:value={searchTerm}
              on:input={searchAdmins}
              class="w-full sm:w-64 h-9 pl-2 border border-gray-400 rounded-md focus:outline-none focus:ring-1 focus:ring-black"
            />
          </div>
      
          <!-- Responsive Table -->
          <table class="min-w-full border-separate border-spacing-0 divide-y divide-gray-200 text-sm">
            <thead class="bg-[#02066F] text-white">
              <tr>
                <th class="px-4 py-3 text-left font-semibold tracking-wide">Pin</th>
                <th class="px-4 py-3 text-left font-semibold tracking-wide">Name</th>
                <th class="px-4 py-3 text-left font-semibold tracking-wide">Phone Number</th>
                <th class="px-4 py-3 text-left font-semibold tracking-wide">Action</th>
              </tr>
            </thead>
      
            <tbody class="bg-white divide-y divide-gray-200">
              {#each admins as admin (admin.pin)}
                <tr class="hover:bg-gray-100">
                  <td class="px-4 py-3 whitespace-nowrap text-gray-900 text-center">{admin.pin}</td>
                  <td class="px-4 py-3 whitespace-nowrap text-gray-900">{admin.name}</td>
                  <td class="px-4 py-3 whitespace-nowrap text-gray-900">{admin.phone}</td>
                  <td class="px-4 py-3 whitespace-nowrap text-gray-900">
                    <div class="flex space-x-6 justify-center text-center">
                      <button on:click={() => openEditEmployee(admin)} class="text-[#02066F] hover:text-[#02066F]">
                        <i class="fas fa-pencil-alt cursor-pointer"></i>
                      </button>
                      <button on:click={() => openDeleteModal(admin)} class="text-[#02066F] hover:text-[#02066F]">
                        <i class="fas fa-trash cursor-pointer "></i>
                      </button>
                    </div>
                  </td>
                </tr>
              {:else}
                <tr>
                  <td colspan="4" class="px-4 py-4 text-center text-sm text-gray-500">No admins found</td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      </div>

      
    <!-- Employee Modal -->
    {#if showEmployeeModal}
        <div 
            class="fixed inset-0 flex items-center justify-center z-50"
            style="background: rgba(0, 0, 0, 0.5)"
            on:click={() => showEmployeeModal = false}
        >
            <div 
                class="bg-white rounded-lg w-full max-w-xs mx-4"
                on:click|stopPropagation
                transition:fade
            >
                <div class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center text-center">
                    <h3 class="text-xl font-semibold p-2 text-white">
                        {isEditing ? 'Admin Details' : 'Employee Details'}
                    </h3>
                    <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2"  on:click={() => showEmployeeModal = false}>
                      ×</button>
                </div>
                <div class="p-6">
                    <form on:submit|preventDefault={submitForm}>
                        <div class="mb-4">
                           
                            <input
                                id="firstName"
                                type="text"
                                placeholder="First Name"
                                bind:value={formData.firstName}
                                class={`w-full px-3 py-2 border-2 ${errors.firstName ? 'border-red-500' : 'border-[#02066F]'} font-bold rounded-md focus:outline-none `}
                            />
                            {#if errors.firstName}
                                <p class="text-red-500 text-xs italic mt-1">{errors.firstName}</p>
                            {/if}
                        </div>
                        <div class="mb-4">
                           
                            <input
                                id="lastName"
                                type="text"
                                placeholder="Last Name"
                                bind:value={formData.lastName}
                                class={`w-full px-3 py-2 border-2 ${errors.lastName ? 'border-red-500' : 'border-[#02066F]'} font-bold rounded-md focus:outline-none`}
                            />
                            {#if errors.lastName}
                                <p class="text-red-500 text-xs italic mt-1">{errors.lastName}</p>
                            {/if}
                        </div>
                        <div class="mb-4">
                           
                            <input
                                id="phone"
                                type="text"
                                placeholder="Phone Number"
                                bind:value={formData.phone}
                                on:input={handlePhoneInput}
                                maxlength="14"
                                class={`w-full px-3 py-2 border-2 ${errors.phone ? 'border-red-500' : 'border-[#02066F]'} font-bold rounded-md focus:outline-none`}
                                
                            />
                            {#if errors.phone}
                                <p class="text-red-500 text-xs italic mt-1">{errors.phone}</p>
                            {/if}
                        </div>
                        <div class="mb-4">
                         
                          <input
                              id="pin"
                              type="text"
                              placeholder="Instructor Pin"
                              bind:value={formData.pin}
                              class="w-full px-3 py-2 border-2 border-[#02066F] font-bold bg-gray-300 rounded-md focus:outline-none "
                              readonly
                          />
                      </div>
                        <div class="flex justify-center mt-6">
                           
                            <button
                                type="submit"
                                class="px-4 py-2 bg-[#02066F] text-white rounded-md cursor-pointer cursor-pointer"
                            >
                                {isEditing ? 'Update' : 'Submit'}
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    {/if}

    <!-- Admin Modal -->
    {#if showAdminModal}
        <div 
            class="fixed inset-0 flex items-center justify-center z-50"
            style="background: rgba(0, 0, 0, 0.5)"
            on:click={() => showAdminModal = false}
        >
            <div 
                class="bg-white rounded-lg w-full max-w-xs mx-4"
                on:click|stopPropagation
                transition:fade
            >
                <div class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center text-center">
                    <h3 class="text-xl font-semibold p-2 text-white">
                        <!-- {isEditing ? 'Admin Details' : 'Admin Details'} -->
                        Admin Details
                    </h3>
                    <button class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2"  on:click={() => showAdminModal = false}>
                      ×</button>
                </div>
                <div class="p-6">
                    <form on:submit|preventDefault={submitForm}>
                      
                        <div class="mb-4">
                            
                            <input
                                id="adminFirstName"
                                type="text"
                                placeholder="First Name"
                                bind:value={formData.firstName}
                                class={`w-full px-3 py-2 font-bold border-2 ${errors.firstName ? 'border-red-500' : 'border-[#02066F]'} rounded-md focus:outline-none `}
                            />
                            {#if errors.firstName}
                                <p class="text-red-500 text-xs italic mt-1">{errors.firstName}</p>
                            {/if}
                        </div>
                        <div class="mb-4">
                            
                            <input
                                id="adminLastName"
                                type="text"
                                placeholder="Last Name"
                                bind:value={formData.lastName}
                                class={`w-full px-3 py-2 font-bold border-2 ${errors.lastName ? 'border-red-500' : 'border-[#02066F]'} rounded-md focus:outline-none`}
                            />
                            {#if errors.lastName}
                                <p class="text-red-500 text-xs italic mt-1">{errors.lastName}</p>
                            {/if}
                        </div>
                        <div class="mb-4">
                           
                            <input
                                id="adminPhone"
                                type="text"
                                bind:value={formData.phone}
                                on:input={handlePhoneInput}
                                maxlength="14"
                                class={`w-full px-3 py-2 font-bold border-2 ${errors.phone ? 'border-red-500' : 'border-[#02066F]'} rounded-md focus:outline-none`}
                                placeholder="Phone Number"
                            />
                            {#if errors.phone}
                                <p class="text-red-500 text-xs italic mt-1">{errors.phone}</p>
                            {/if}
                        </div>
                        <div class="mb-4">
                          <input
                              id="adminPin"
                              type="text"
                              placeholder="Instructor Pin"
                              bind:value={formData.pin}
                              class="w-full px-3 py-2 font-bold border-2 border-[#02066F] rounded-md bg-gray-300 focus:outline-none"
                              readonly
                          />
                      </div>
                        <div class="flex justify-center mt-6">
                            
                            <button
                                type="submit"
                                class="px-4 py-2 bg-[#02066F] text-white rounded-md cursor-pointer"
                            >
                                {isEditing ? 'Update' : 'Submit'}
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    {/if}


    
    <!-- Delete Confirmation Modal -->
    {#if showDeleteModal}
        <div 
            class="fixed inset-0 flex items-center justify-center z-50"
            style="background: rgba(0, 0, 0, 0.5)"
            role="dialog"
            aria-modal="true"
            tabindex="-1"
            on:click={() => showDeleteModal = false}
            on:keydown={(e) => { if (e.key === 'Escape') showDeleteModal = false; }}
        >
            <div 
                class="bg-white rounded-lg w-full max-w-md mx-4"
                on:click|stopPropagation
                transition:fade
            >
                <div class="bg-[#02066F] text-white p-4 rounded-t-lg">
                    <h3 class="text-lg font-semibold text-center">Delete</h3>
                </div>
                <div class="p-6">
                    {#if currentEmployee}
                        <p class="text-center mb-6">
                            Are you sure you want to remove this {currentEmployee.isAdmin ? 'admin' : 'employee'}?
                        </p>
                    {:else}
                        <p class="text-center mb-6">
                            Are you sure you want to remove this employee?
                        </p>
                    {/if}
                    <div class="flex justify-center space-x-4">
                        <button
                            type="button"
                            on:click={() => showDeleteModal = false}
                            class="px-4 py-2 border border-[#02066F] text-[#02066F] rounded-md hover:bg-gray-100 cursor-pointer"
                        >
                            No
                        </button>
                        <button
                            type="button"
                            on:click={deleteEmployee}
                            class="px-4 py-2 bg-[#02066F] text-white rounded-md hover:bg-blue-800 cursor-pointer"
                        >
                            Yes
                        </button>
                    </div>
                </div>
            </div>
        </div>
    {/if}
</div>

<style>
    /* Import Font Awesome for icons */
    @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css');
    
    /* Custom styles for the table */
    table {
        width: 100%;
        border-collapse: separate;
        border-spacing: 0;
    }
    
    th, td {
        padding: 12px 15px;
        text-align: left;
        border-bottom: 1px solid #e5e7eb;
        border-right: 1px solid #d3d3d3;
        border-left: 1px solid #d3d3d3;
        text-align: center;
    }
    
    th {
        padding: 15px;
        background-color: #02066F;
        color: white;
        font-weight: bold;
        font-size: rem;
        letter-spacing: 0.05em;
    }
    
    tr:nth-child(even) {
        background-color: #f9fafb;
    }
    
    tr:hover {
        background-color: #f3f4f6;
    }
    
    /* Responsive table */
    @media (max-width: 640px) {
        table {
            display: block;
            overflow-x: auto;
            white-space: nowrap;
        }
    }
    
    /* Modal animations */
    .modal-enter {
        opacity: 0;
    }
    
    .modal-enter-active {
        opacity: 1;
        transition: opacity 200ms;
    }
    
    .modal-exit {
        opacity: 1;
    }
    
    .modal-exit-active {
        opacity: 0;
        transition: opacity 200ms;
    }
</style>