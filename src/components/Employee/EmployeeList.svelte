<script lang="ts">
  import { onMount } from "svelte";
  import { fade } from "svelte/transition";
  import { v4 as uuidv4 } from "uuid";

  type Employee = {
    EmpID: string;
    Pin: string;
    FName: string;
    LName: string;
    PhoneNumber: string;
    IsAdmin: number; // 0 = employee, 1 = admin, 2 = superadmin
    Email?: any;
    IsActive: boolean;
    DeviceID?: string; // Added DeviceID property
  };

  // Data state
  let employees: Employee[] = [];
  let filteredEmployees: Employee[] = [];
  let admins: Employee[] = [];
  let superAdmins: Employee[] = [];
  let devices: any[] = [];
  let selectedDevice: any = null;
  let searchTerm: string = "";
  let searchTerms: string = "";
  let searchSuperAdminTerm: string = "";
  let getEmail: string = "";

  // UI state
  let showEmployeeModal = false;
  let showAdminModal = false;
  let showSuperAdminModal = false;
  let showDeleteModal = false;
  let currentEmployee: Employee | null = null;
  let isEditing = false;
  let loading = true;
  let adminCount = 0;
  let superAdminCount = 0;

  // API config
  const apiUrlBase =
    "https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/employee";
  const deviceApiUrl =
    "https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/device";
  const adminType = localStorage.getItem("adminType");
  const deviceID = localStorage.getItem("DeviceID");
  // Form data
  let formData = {
    EmpID: "",
    Pin: "",
    FName: "",
    LName: "",
    PhoneNumber: "",
    Email: "",
    IsAdmin: 0,
    IsActive: true,
    LastModifiedBy: "Admin",
  };

  // Validation errors
  let errors = {
    FName: "",
    LName: "",
    PhoneNumber: "",
    Email: "",
  };

  // Success/error messages
  let successMessage = "";
  let errorMessage = "";

  // Sorting and pagination state
  let sortConfig = {
    key: "Pin",
    direction: "asc",
  };

  let pageSize = 10;
  let currentPage = 1;
  let pageSizeOptions = [10, 25, 50, 100];
  let paginatedEmployees: Employee[] = [];

  let userType = "";

  // Initialize component



  onMount(async () => {
    getEmail = localStorage.getItem('adminMail') || "";
    await fetchEmployeeData();
    await fetchDevices();


  });
  onMount(() => {
    fetchEmployeeData();
    // userType = localStorage.getItem('userType') || '';
    userType = localStorage.getItem('adminType') || '';


  });

  // Fetch all employee data
  async function fetchEmployeeData() {
    try {
      loading = true;
      const companyId = localStorage.getItem("companyID");
      const response = await fetch(`${apiUrlBase}/getall/${companyId}`);

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
      }

      const data = await response.json();
      employees = data;
      filterEmployees();
    } catch (error) {
      console.error("Error fetching employee data:", error);
      errorMessage = "Failed to load employee data";
      setTimeout(() => (errorMessage = ""), 3000);
    } finally {
      loading = false;
    }
  }

  // Fetch device data
  async function fetchDevices() {
    try {
      const companyId = localStorage.getItem("companyID");
      const response = await fetch(`${deviceApiUrl}/getAll/${companyId}`);

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
      }

      const data = await response.json();
      // Filter out devices with "Not Registered" names
      const allDevices = Array.isArray(data) ? data : [data];
      devices = allDevices.filter(
        (device) =>
          device.DeviceName &&
          device.DeviceName !== "Not Registered" &&
          device.DeviceName.trim() !== "",
      );
      console.log("Fetched devices (filtered):", devices);

      // Set first valid device as default selection
      if (devices.length > 0) {
        selectedDevice = devices[0];
        console.log("Default selected device:", selectedDevice);
        console.log("Default Device ID:", selectedDevice.DeviceID);
        // Filter employees after setting default device
        filterEmployees();
      }
    } catch (error) {
      console.error("Error fetching devices:", error);
      errorMessage = "Failed to load devices";
      setTimeout(() => (errorMessage = ""), 3000);
    }
  }

  // Handle device selection
  function handleDeviceSelection(device: any) {
    selectedDevice = device;
    console.log("Selected device:", device);
    console.log("Device ID to pass:", device.DeviceID);
    // Filter employees by selected device
    filterEmployees();
  }

  // Filter employees by type and device
  function filterEmployees() {
    loading = true;

    // Base filter for device selection
    let deviceFilteredEmployees = employees;
    if (selectedDevice && selectedDevice.DeviceID) {
      deviceFilteredEmployees = employees.filter(
        (emp) => emp.DeviceID === selectedDevice.DeviceID,
      );
      console.log(
        `Filtered employees by DeviceID ${selectedDevice.DeviceID}:`,
        deviceFilteredEmployees.length,
      );
    }

    // Filter by employee type from device-filtered employees
    filteredEmployees = deviceFilteredEmployees.filter(
      (emp) => emp.IsAdmin === 0,
    );
    admins = deviceFilteredEmployees.filter((emp) => emp.IsAdmin === 1);
    superAdmins = deviceFilteredEmployees.filter((emp) => emp.IsAdmin === 2);

    adminCount = admins.length;
    superAdminCount = superAdmins.length;
    updatePagination();
    loading = false;

    // Profile page Admin and Super Admin details storing

    let matchedEmployee = null;
    const cleanEmail = getEmail.trim().toLowerCase();

    console.log("okay",admins);

    if (adminType === "Admin") {
      for (const emp of admins) {
        const empEmail = (emp.Email || "").trim().toLowerCase();
        console.log(`Comparing Admin Email: ${empEmail} === ${cleanEmail}`);
        if (empEmail === cleanEmail) {
          matchedEmployee = emp;
          break;
        }
      }
    } else if (adminType === "SuperAdmin") {
      for (const emp of superAdmins) {
        const empEmail = (emp.Email || "").trim().toLowerCase();
        console.log(
          `Comparing SuperAdmin Email: ${empEmail} === ${cleanEmail}`,
        );
        if (empEmail === cleanEmail) {
          matchedEmployee = emp;
          break;
        }
      }
    }

    console.log("Matched Employee:", matchedEmployee);

    if (matchedEmployee) {
      localStorage.setItem("loggedAdmin", JSON.stringify(matchedEmployee));
    } else {
      console.log("No matching employee found.");
    }
  }

  // Search functions
  function searchEmployees() {
    if (!searchTerms) {
      filterEmployees();
      return;
    }

    // Base filter for device selection
    let deviceFilteredEmployees = employees;
    if (selectedDevice && selectedDevice.DeviceID) {
      deviceFilteredEmployees = employees.filter(
        (emp) =>
          emp.DeviceID ===
          (deviceID != undefined || deviceID == null
            ? selectedDevice.DeviceID
            : deviceID),
      );
    }

    const term = searchTerms.toLowerCase();
    filteredEmployees = deviceFilteredEmployees.filter(
      (emp) =>
        emp.IsAdmin === 0 &&
        (emp.FName.toLowerCase().includes(term) ||
          emp.LName.toLowerCase().includes(term) ||
          emp.Pin.includes(searchTerms) ||
          emp.PhoneNumber.includes(searchTerms)),
    );
    updatePagination();
  }

  function searchAdmins() {
    if (!searchTerm) {
      filterEmployees();
      return;
    }

    // Base filter for device selection
    let deviceFilteredEmployees = employees;
    if (selectedDevice && selectedDevice.DeviceID) {
      deviceFilteredEmployees = employees.filter(
        (emp) =>
          emp.DeviceID ===
          (deviceID != undefined || deviceID == null
            ? selectedDevice.DeviceID
            : deviceID),
      );
    }

    const term = searchTerm.toLowerCase();
    admins = deviceFilteredEmployees.filter(
      (emp) =>
        emp.IsAdmin === 1 &&
        (emp.FName.toLowerCase().includes(term) ||
          emp.LName.toLowerCase().includes(term) ||
          emp.Pin.includes(searchTerm) ||
          emp.PhoneNumber.includes(searchTerm)),
    );
  }

  function searchSuperAdmins() {
    if (!searchSuperAdminTerm) {
      filterEmployees();
      return;
    }

    // Base filter for device selection
    let deviceFilteredEmployees = employees;
    if (selectedDevice && selectedDevice.DeviceID) {
      deviceFilteredEmployees = employees.filter(
        (emp) =>
          emp.DeviceID ===
          (deviceID != undefined || deviceID == null
            ? selectedDevice.DeviceID
            : deviceID),
      );
    }

    const term = searchSuperAdminTerm.toLowerCase();
    superAdmins = deviceFilteredEmployees.filter(
      (emp) =>
        emp.IsAdmin === 2 &&
        (emp.FName.toLowerCase().includes(term) ||
          emp.LName.toLowerCase().includes(term) ||
          emp.Pin.includes(searchSuperAdminTerm) ||
          emp.PhoneNumber.includes(searchSuperAdminTerm)),
    );
  }

  // Sorting function
  function requestSort(key: string) {
    let direction = "asc";
    if (sortConfig.key === key && sortConfig.direction === "asc") {
      direction = "desc";
    }
    sortConfig = { key, direction };

    filteredEmployees = [...filteredEmployees].sort((a, b) => {
      //@ts-ignore
      if (a[key] < b[key]) return direction === "asc" ? -1 : 1;
      //@ts-ignore
      if (a[key] > b[key]) return direction === "asc" ? 1 : -1;
      return 0;
    });
    updatePagination();
  }

  // Pagination functions
  function updatePagination() {
    const start = (currentPage - 1) * pageSize;
    const end = start + pageSize;
    paginatedEmployees = filteredEmployees.slice(start, end);
    const employeeData = paginatedEmployees.map((emp) => ({
      id: emp.EmpID,
      name: `${emp.FName} ${emp.LName}`,
    }));

    localStorage.setItem("employeeData", JSON.stringify(employeeData));
  }

  function handlePageSizeChange(newSize: number) {
    pageSize = newSize;
    currentPage = 1;
    updatePagination();
  }

  // Format phone number display
  function formatPhoneNumber(phone: string): string {
    if (!phone) return "";
    let value = phone.replace(/\D/g, "");
    if (value.length > 6) {
      value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6, 10)}`;
    } else if (value.length > 3) {
      value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
    } else {
      value = `(${value}`;
    }
    return value;
  }

  // Handle phone input with auto-formatting
  function handlePhoneInput(e: Event) {
    const target = e.target as HTMLInputElement;
    let value = target.value.replace(/\D/g, "");
    if (value.length > 10) value = value.slice(0, 10);

    if (value.length > 6) {
      value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6)}`;
    } else if (value.length > 3) {
      value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
    } else if (value.length > 0) {
      value = `(${value}`;
    }

    formData.PhoneNumber = value;

    // Auto-generate PIN from last 4 digits
    if (value.length >= 4) {
      formData.Pin = value.slice(-4);
    }
  }

  // Form validation
  function validateForm(): boolean {
    let isValid = true;
    errors = { FName: "", LName: "", PhoneNumber: "", Email: "" };

    // First name validation
    if (!formData.FName.trim()) {
      errors.FName = "First name is required";
      isValid = false;
    } else if (!/^[a-zA-Z\s]+$/.test(formData.FName)) {
      errors.FName = "Only letters allowed";
      isValid = false;
    }

    // Last name validation
    if (!formData.LName.trim()) {
      errors.LName = "Last name is required";
      isValid = false;
    } else if (!/^[a-zA-Z\s]+$/.test(formData.LName)) {
      errors.LName = "Only letters allowed";
      isValid = false;
    }

    // Phone validation
    const phoneRegex = /^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$/;
    if (!formData.PhoneNumber) {
      errors.PhoneNumber = "Phone number is required";
      isValid = false;
    } else if (!phoneRegex.test(formData.PhoneNumber)) {
      errors.PhoneNumber = "Invalid phone number format";
      isValid = false;
    }

    // Email validation for admins/superadmins
    if (formData.IsAdmin > 0) {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!formData.Email) {
        errors.Email = "Email is required for admin";
        isValid = false;
      } else if (!emailRegex.test(formData.Email)) {
        errors.Email = "Invalid email format";
        isValid = false;
      }
    }

    return isValid;
  }

  // Modal open functions
  function openAddEmployee() {
    console.log("openAddEmployee called");
    formData = {
      EmpID: "",
      Pin: "",
      FName: "",
      LName: "",
      PhoneNumber: "",
      Email: "",
      IsAdmin: 0,
      IsActive: true,
      LastModifiedBy: "Admin",
    };
    errors = { FName: "", LName: "", PhoneNumber: "", Email: "" };
    successMessage = "";
    errorMessage = "";
    isEditing = false;
    showEmployeeModal = true;
    console.log("Employee modal opened, isEditing:", isEditing);
  }

  function openAddAdmin() {
    console.log("openAddAdmin called, adminCount:", adminCount);
    if (adminCount >= 3) {
      errorMessage = "Maximum 3 admins allowed";
      setTimeout(() => (errorMessage = ""), 3000);
      return;
    }

    formData = {
      EmpID: "",
      Pin: "",
      FName: "",
      LName: "",
      PhoneNumber: "",
      Email: "",
      IsAdmin: 1,
      IsActive: true,
      LastModifiedBy: "Admin",
    };
    errors = { FName: "", LName: "", PhoneNumber: "", Email: "" };
    successMessage = "";
    errorMessage = "";
    isEditing = false;
    showAdminModal = true;
    console.log("Admin modal opened, isEditing:", isEditing);
  }

  function openAddSuperAdmin() {
    console.log("openAddSuperAdmin called");
    formData = {
      EmpID: "",
      Pin: "",
      FName: "",
      LName: "",
      PhoneNumber: "",
      Email: "",
      IsAdmin: 2,
      IsActive: true,
      LastModifiedBy: "Admin",
    };
    errors = { FName: "", LName: "", PhoneNumber: "", Email: "" };
    successMessage = "";
    errorMessage = "";
    isEditing = false;
    showSuperAdminModal = true;
    console.log("SuperAdmin modal opened, isEditing:", isEditing);
  }

  function openEditEmployee(employee: Employee) {
    currentEmployee = employee;
    formData = {
      EmpID: employee.EmpID,
      Pin: employee.Pin,
      FName: employee.FName,
      LName: employee.LName,
      PhoneNumber: employee.PhoneNumber,
      Email: employee.Email || "",
      IsAdmin: employee.IsAdmin,
      IsActive: employee.IsActive,
      LastModifiedBy: "Admin",
    };
    isEditing = true;

    if (employee.IsAdmin === 0) {
      showEmployeeModal = true;
    } else if (employee.IsAdmin === 1) {
      showAdminModal = true;
    } else {
      showSuperAdminModal = true;
    }
  }

  function openDeleteModal(employee: Employee) {
    currentEmployee = employee;
    showDeleteModal = true;
  }

  // Form submission
  async function submitForm() {
    if (!validateForm()) return;

    // Validate device selection
    if (!selectedDevice) {
      errorMessage = "Please select a device before adding employee";
      setTimeout(() => (errorMessage = ""), 3000);
      return;
    }

    try {
      loading = true;
      const companyId = localStorage.getItem("companyID");
      const employeeData = {
        ...formData,
        CID: companyId,
        DeviceID:
          adminType != "Owner"
            ? deviceID
            : selectedDevice
              ? selectedDevice.DeviceID
              : null,
      };

      // Generate UUID for new employee creation
      if (!isEditing) {
        employeeData.EmpID = uuidv4();
        console.log("Generated EmpID for new employee:", employeeData.EmpID);
      }

      // Convert empty email string to null for both create and update
      if (employeeData.Email === "" || employeeData.Email === undefined) {
        employeeData.Email = "";
      }

      console.log("Submitting employee data with DeviceID:", employeeData);

      const apiUrl = isEditing
        ? `${apiUrlBase}/update/${formData.EmpID}`
        : `${apiUrlBase}/create`;
      const method = isEditing ? "PUT" : "POST";

      const response = await fetch(apiUrl, {
        method,
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(employeeData),
      });

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
      }

      const data = await response.json();
      if (data.error) {
        errorMessage = data.error;
      } else {
        successMessage = isEditing
          ? "Employee updated successfully"
          : "Employee added successfully";
        console.log("Success! Refreshing employee data...");
        fetchEmployeeData();
      }
    } catch (error) {
      console.error("Error submitting form:", error);
      errorMessage = "An error occurred while saving the data";
    } finally {
      loading = false;
      setTimeout(() => {
        successMessage = "";
        errorMessage = "";
      }, 3000);

      if (!errorMessage) {
        showEmployeeModal = false;
        showAdminModal = false;
        showSuperAdminModal = false;
      }
    }
  }

  // Delete employee
  async function deleteEmployee() {
    if (!currentEmployee) return;

    try {
      loading = true;
      const response = await fetch(
        `${apiUrlBase}/delete/${currentEmployee.EmpID}/Admin`,
        {
          method: "PUT",
        },
      );

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
      }

      const data = await response.json();
      if (data.error) {
        errorMessage = data.error;
      } else {
        successMessage = "Employee deleted successfully";
        fetchEmployeeData();
      }
    } catch (error) {
      console.error("Error deleting employee:", error);
      errorMessage = "An error occurred while deleting the employee";
    } finally {
      loading = false;
      setTimeout(() => {
        successMessage = "";
        errorMessage = "";
      }, 3000);
      showDeleteModal = false;
    }
  }

  // DROPDOWN DEVICE CLICK BODY ACTION 
  let dropdownOpen = false;

  function toggleDropdown() {
    dropdownOpen = !dropdownOpen;
  }

  // Close dropdown when clicking outside
  function handleClickOutside(event: MouseEvent) {
    const dropdown = document.getElementById("device-dropdown-summary");
    const button = document.getElementById("device-menu-button-summary");

    if (
      dropdown &&
      !dropdown.contains(event.target as Node) &&
      button &&
      !button.contains(event.target as Node)
    ) {
      dropdownOpen = false;
    }
  }

  onMount(() => {
    window.addEventListener("click", handleClickOutside);
    return () => {
      window.removeEventListener("click", handleClickOutside);
    };
  });

  function selectDevice(device: any) {
    handleDeviceSelection(device);
    dropdownOpen = false;
  }
  //END DROPDOWN DEVICE CLICK BODY ACTION
  
</script>

<div class="min-h-screen bg-gray-100 pt-30 pb-10 px-4 sm:px-6">
  <!-- Loading Overlay -->
  {#if loading}
    <div
      class="fixed inset-0 flex items-center justify-center z-50"
      style="background: rgba(0, 0, 0, 0.5)"
    >
      <div
        class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"
      ></div>
    </div>
  {/if}

  <!-- Messages -->
  {#if successMessage}
    <div
      class="fixed top-4 right-4 z-50 bg-green-500 text-white px-4 py-2 rounded-md shadow-lg"
    >
      {successMessage}
    </div>
  {/if}

  {#if errorMessage}
    <div
      class="fixed top-4 right-4 z-50 bg-red-500 text-white px-4 py-2 rounded-md shadow-lg"
    >
      {errorMessage}
    </div>
  {/if}

   <!-- Device Dropdown Section -->
    <div class="max-w-5xl mx-auto mb-8 px-4">
      <div class="flex justify-center">
        <div class="relative inline-block text-left w-64">
          {#if adminType === "Owner"}
            <button
              id="device-menu-button-summary"
              type="button"
              class="inline-flex w-full justify-between items-center rounded-lg bg-white px-4 py-3 text-sm font-semibold text-[#02066F] border border-[#02066F] shadow-sm hover:bg-[#02066F] hover:text-white focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-[#02066F] transition"
              on:click={toggleDropdown}
            >
              <span
                >{selectedDevice
                  ? selectedDevice.DeviceName
                  : "Select Device Name"}</span
              >
              <svg
                class="h-5 w-5 text-gray-400 group-hover:text-white transition"
                viewBox="0 0 20 20"
                fill="currentColor"
                aria-hidden="true"
              >
                <path
                  fill-rule="evenodd"
                  d="M5.23 7.21a.75.75 0 011.06.02L10 11.168l3.71-3.938a.75.75 0 111.08 1.04l-4.25 4.5a.75.75 0 01-1.08 0l-4.25-4.5a.75.75 0 01.02-1.06z"
                  clip-rule="evenodd"
                />
              </svg>
            </button>
          {/if}

          <!-- Dropdown -->
          {#if dropdownOpen}
            <div
              id="device-dropdown-summary"
              class="absolute right-0 z-20 mt-2 w-full origin-top-right rounded-lg bg-white shadow-lg ring-1 ring-black ring-opacity-5 animate-fadeIn"
            >
              <div class="py-1">
                {#each devices as device}
                  <button
                    type="button"
                    class="text-gray-700 block w-full px-4 py-2 text-left text-sm hover:bg-[#02066F] hover:text-white transition"
                    on:click={() => selectDevice(device)}
                  >
                    {device.DeviceName}
                  </button>
                {:else}
                  <div class="text-gray-500 block px-4 py-2 text-sm">
                    No devices available
                  </div>
                {/each}
              </div>
            </div>
          {/if}
        </div>
      </div>
    </div>
    <!-- End Device Dropdown -->

  <!-- Employee Section -->
  <div class="max-w-5xl mx-auto">
    <div class="mb-8">
      <div
        class="flex flex-row sm:flex-row justify-between items-start sm:items-center mb-4 gap-4"
      >
        <h2 class="text-2xl sm:text-3xl font-bold text-gray-800">
          Employee Details
        </h2>
        <button
          on:click={openAddEmployee}
          class="px-2 py-1 md:px-2 md:py-1 w-24 h-10 md:w-26 text-center items-center bg-white text-base md:text-lg border border-[#02066F] text-[#02066F] rounded-md cursor-pointer transition-colors"
        >
          Add Entry
        </button>
      </div>

      <div
        class="bg-white rounded-xl shadow-md overflow-hidden border border-gray-300"
      >
        <div class="p-4 sm:p-6 overflow-x-auto">
          <div
            class="flex flex-col sm:flex-row justify-between items-star sm:items-center mb-4 gap-4"
          >
            <div
              class="flex items-center space-x-2 md:mt-[-16px] text-center justify-center"
            >
              <span class="text-base font-semibold text-gray-700">Show</span>
              <select
                class="border border-gray-400 rounded-md px-2 py-1 text-sm focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                bind:value={pageSize}
                on:change={() => {
                  handlePageSizeChange(pageSize);
                  currentPage = 1;
                }}
              >
                {#each pageSizeOptions as option}
                  <option value={option}>{option}</option>
                {/each}
              </select>
              <span class="text-base font-semibold text-gray-700">entries</span>
            </div>

            <div
              class="mb-4 flex flex-col sm:flex-row sm:items-center sm:justify-end gap-2"
            >
              <label class="text-base font-semibold text-gray-800"
                >Search:</label
              >
              <input
                type="text"
                bind:value={searchTerms}
                on:input={searchEmployees}
                class="w-full sm:w-64 px-2 py-1 border border-gray-500 rounded-md focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                placeholder=""
              />
            </div>
          </div>

          <div class="overflow-x-auto">
            <table class="min-w-full divide-y divide-gray-200">
              <thead class="bg-[#02066F] text-white">
                <tr>
                  <th
                    class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer"
                    on:click={() => requestSort("Pin")}
                  >
                    <!-- <div class="flex items-center"> -->
                    Pin
                    {#if sortConfig.key === "Pin"}
                      <span class="ml-6 text-lg"
                        >{sortConfig.direction === "asc" ? "↑" : "↓"}</span
                      >
                    {:else}
                      <span class="ml-6 text-lg">↑↓</span>
                    {/if}
                    <!-- </div> -->
                  </th>
                  <th
                    class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer"
                    on:click={() => requestSort("FName")}
                  >
                    <!-- <div class="flex items-center"> -->
                    Name
                    {#if sortConfig.key === "FName"}
                      <span class="ml-6 text-lg"
                        >{sortConfig.direction === "asc" ? "↑" : "↓"}</span
                      >
                    {:else}
                      <span class="ml-6 text-lg">↑↓</span>
                    {/if}
                    <!-- </div> -->
                  </th>
                  <th
                    class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer"
                    on:click={() => requestSort("PhoneNumber")}
                  >
                    <!-- <div class="flex items-center"> -->
                    Phone Number
                    {#if sortConfig.key === "PhoneNumber"}
                      <span class="ml-6 text-lg"
                        >{sortConfig.direction === "asc" ? "↑" : "↓"}</span
                      >
                    {:else}
                      <span class="ml-6 text-lg">↑↓</span>
                    {/if}
                    <!-- </div> -->
                  </th>
                  <th
                    class="px-6 py-3 text-center text-base font-bold tracking-wider"
                  >
                    Action
                  </th>
                </tr>
              </thead>
              <tbody class="bg-white divide-y divide-gray-200">
                {#each paginatedEmployees as employee (employee.EmpID)}
                  <tr class="hover:bg-gray-50">
                    <td
                      class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                      >{employee.Pin}</td
                    >
                    <td
                      class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                      >{employee.FName} {employee.LName}</td
                    >
                    <td
                      class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                      >{formatPhoneNumber(employee.PhoneNumber)}</td
                    >
                    <td
                      class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                    >
                      <div class="flex justify-center space-x-2">
                        <button
                          on:click={() => openEditEmployee(employee)}
                          class="text-[#02066F] p-1 cursor-pointer"
                        >
                          <i class="fas fa-pencil-alt"></i>
                        </button>
                        <button
                          on:click={() => openDeleteModal(employee)}
                          class="text-[#02066F] p-1 cursor-pointer"
                        >
                          <i class="fas fa-trash"></i>
                        </button>
                      </div>
                    </td>
                  </tr>
                {:else}
                  <tr>
                    <td
                      colspan="4"
                      class="px-4 py-4 text-center text-sm text-gray-500"
                      >No employees found</td
                    >
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>

          <!-- Pagination controls -->
          <div
            class="flex flex-col sm:flex-row items-center justify-between mt-4"
          >
            <div class="text-base font-semibold text-gray-700 mb-2 sm:mb-0">
              Showing {Math.min(
                (currentPage - 1) * pageSize + 1,
                filteredEmployees.length,
              )} to
              {Math.min(currentPage * pageSize, filteredEmployees.length)} of
              {filteredEmployees.length} entries
            </div>
            <div class="flex space-x-1">
              <button
                class="px-3 py-1 rounded-md text-base font-semibold text-gray-500 disabled:opacity-50"
                disabled={currentPage === 1}
                on:click={() => {
                  currentPage--;
                  updatePagination();
                }}
              >
                Previous
              </button>
              <button
                class="px-3 py-1 rounded-md text-base font-semibold text-gray-500 disabled:opacity-50"
                disabled={currentPage * pageSize >= filteredEmployees.length}
                on:click={() => {
                  currentPage++;
                  updatePagination();
                }}
              >
                Next
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Admin Section (only for SuperAdmin) -->

    {#if adminType != "Admin"}

    <!-- {#if adminType === 'SuperAdmin'} -->
     {#if adminType !== 'Admin'}
      <div class="mb-8 pt-4">
        <div
          class="flex flex-row sm:flex-row justify-between items-start sm:items-center mb-4 gap-4"
        >
          <h2 class="text-2xl sm:text-3xl font-bold text-gray-800">
            Admin Details
          </h2>
          <button
            on:click={openAddAdmin}
            disabled={adminCount >= 3}
            class="px-2 py-1 md:px-2 md:py-1 w-24 h-10 md:w-26 text-center items-center bg-white text-base md:text-lg border border-[#02066F] text-[#02066F] rounded-md cursor-pointer transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
          >
            Add Entry {adminCount >= 3 ? "(Max Reached)" : ""}
          </button>
        </div>

        <div
          class="bg-white rounded-xl shadow-md overflow-hidden border border-gray-300"
        >
          <div class="p-4 sm:p-6 overflow-x-auto">
            <div
              class="mb-4 flex flex-col sm:flex-row sm:items-center sm:justify-end gap-2"
            >
              <label class="text-base font-semibold text-gray-800"
                >Search:</label
              >
              <input
                type="text"
                bind:value={searchTerm}
                on:input={searchAdmins}
                class="w-full sm:w-64 px-2 py-1 border border-gray-500 rounded-md focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                placeholder=""
              />
            </div>

            <div class="overflow-x-auto">
              <table class="min-w-full divide-y divide-gray-200">
                <thead class="bg-[#02066F] text-white">
                  <tr>
                    <th
                      class="px-4 py-3 text-left text-base font-bold tracking-wider"
                      >Pin</th
                    >
                    <th
                      class="px-4 py-3 text-left text-base font-bold tracking-wider"
                      >Name</th
                    >
                    <th
                      class="px-4 py-3 text-left text-base font-bold tracking-wider"
                      >Phone Number</th
                    >
                    <th
                      class="px-4 py-3 text-left text-base font-bold tracking-wider"
                      >Action</th
                    >
                  </tr>
                </thead>
                <tbody class="bg-white divide-y divide-gray-200">
                  {#each admins as admin (admin.EmpID)}
                    <tr class="hover:bg-gray-50">
                      <td
                        class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                        >{admin.Pin}</td
                      >
                      <td
                        class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                        >{admin.FName} {admin.LName}</td
                      >
                      <td
                        class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                        >{formatPhoneNumber(admin.PhoneNumber)}</td
                      >
                      <td
                        class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                      >
                        <div class="flex justify-center space-x-2">
                          <button
                            on:click={() => openEditEmployee(admin)}
                            class="text-[#02066F] p-1 cursor-pointer"
                          >
                            <i class="fas fa-pencil-alt"></i>
                          </button>
                          <button
                            on:click={() => openDeleteModal(admin)}
                            class="text-[#02066F] p-1 cursor-pointer"
                          >
                            <i class="fas fa-trash"></i>
                          </button>
                        </div>
                      </td>
                    </tr>
                  {:else}
                    <tr>
                      <td
                        colspan="4"
                        class="px-4 py-4 text-center text-sm text-gray-500"
                        >No admins found</td
                      >
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
      {/if}

      <!-- SuperAdmin Section -->



       {#if adminType === "Owner"}

      <div class="mb-8 pt-4">
        <div
          class="flex flex-row sm:flex-row justify-between items-start sm:items-center mb-4 gap-4"
        >
          <h2 class="text-2xl sm:text-3xl font-bold text-gray-800">
            SuperAdmin Details
          </h2>
          <button
            on:click={openAddSuperAdmin}
            class="px-2 py-1 md:px-2 md:py-1 w-26 h-10 md:w-26 text-center items-center bg-white text-base md:text-lg border border-[#02066F] text-[#02066F] rounded-md cursor-pointer transition-colors"
          >
            Add Entry
          </button>
        </div>

        <div
          class="bg-white rounded-xl shadow-md overflow-hidden border border-gray-300"
        >
          <div class="p-4 sm:p-6 overflow-x-auto">
            <div
              class="mb-4 flex flex-col sm:flex-row sm:items-center sm:justify-end gap-2"
            >
              <label class="text-base font-semibold text-gray-800"
                >Search:</label
              >
              <input
                type="text"
                bind:value={searchSuperAdminTerm}
                on:input={searchSuperAdmins}
                class="w-full sm:w-64 px-2 py-1 border border-gray-500 rounded-md focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                placeholder=""
              />
            </div>

            <div class="overflow-x-auto">
              <table class="min-w-full divide-y divide-gray-200">
                <thead class="bg-[#02066F] text-white">
                  <tr>
                    <th
                      class="px-4 py-3 text-left text-base font-bold tracking-wider"
                      >Pin</th
                    >
                    <th
                      class="px-4 py-3 text-left text-base font-bold tracking-wider"
                      >Name</th
                    >
                    <th
                      class="px-4 py-3 text-left text-base font-bold tracking-wider"
                      >Phone Number</th
                    >
                    <th
                      class="px-4 py-3 text-left text-base font-bold tracking-wider"
                      >Action</th
                    >
                  </tr>
                </thead>
                <tbody class="bg-white divide-y divide-gray-200">
                  {#each superAdmins as superAdmin (superAdmin.EmpID)}
                    <tr class="hover:bg-gray-50">
                      <td
                        class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 text-center"
                        >{superAdmin.Pin}</td
                      >
                      <td
                        class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                        >{superAdmin.FName} {superAdmin.LName}</td
                      >
                      <td
                        class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                        >{formatPhoneNumber(superAdmin.PhoneNumber)}</td
                      >
                      <td
                        class="px-4 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
                      >
                        <div class="flex justify-center space-x-2">
                          <button
                            on:click={() => openEditEmployee(superAdmin)}
                            class="text-[#02066F] p-1 cursor-pointer"
                          >
                            <i class="fas fa-pencil-alt"></i>
                          </button>
                          <button
                            on:click={() => openDeleteModal(superAdmin)}
                            class="text-[#02066F] p-1 cursor-pointer"
                          >
                            <i class="fas fa-trash"></i>
                          </button>
                        </div>
                      </td>
                    </tr>
                  {:else}
                    <tr>
                      <td
                        colspan="4"
                        class="px-4 py-4 text-center text-sm text-gray-500"
                        >No SuperAdmins found</td
                      >
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>


      {/if}
    {/if}

  </div>

  <!-- Employee Modal -->
  {#if showEmployeeModal}
    <div
      class="fixed inset-0 flex items-center justify-center z-50"
      style="background: rgba(0, 0, 0, 0.5)"
      on:click={() => (showEmployeeModal = false)}
    >
      <div
        class="bg-white rounded-lg w-full max-w-xs mx-4"
        on:click|stopPropagation
        transition:fade
      >
        <div
          class="flex w-full bg-[#02066F] justify-between p-3 pl-4 pr-4 items-center rounded-t-md text-center"
        >
          <h3 class="text-xl font-bold text-white">Employee Details</h3>
          <button
            class="text-gray-400 hover:text-white cursor-pointer text-5xl"
            on:click={() => (showEmployeeModal = false)}
          >
            &times;
          </button>
        </div>
        <div class="p-6">
          <!-- Device Info Display -->
          {#if selectedDevice}
            <div class="mb-4 p-2 bg-blue-50 border border-blue-200 rounded-md">
              <p class="text-sm text-blue-800">
                <strong>Device:</strong>
                {selectedDevice.DeviceName}
              </p>
            </div>
          {/if}

          <form on:submit|preventDefault={submitForm}>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="text"
                bind:value={formData.FName}
                class={`w-full px-3 py-2 font-bold border-2 ${errors.FName ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none focus:ring-1 focus:ring-[#02066F]`}
                placeholder="First Name"
              />
              {#if errors.FName}
                <p class="mt-1 text-sm text-red-600">{errors.FName}</p>
              {/if}
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="text"
                bind:value={formData.LName}
                class={`w-full px-3 py-2 font-bold border-2 ${errors.LName ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none focus:ring-1 focus:ring-[#02066F]`}
                placeholder="Last Name"
              />
              {#if errors.LName}
                <p class="mt-1 text-sm text-red-600">{errors.LName}</p>
              {/if}
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="text"
                bind:value={formData.PhoneNumber}
                on:input={handlePhoneInput}
                maxlength="14"
                class={`w-full px-3 py-2 border-2 ${errors.PhoneNumber ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none focus:ring-1 focus:ring-[#02066F] font-bold`}
                placeholder="Phone Number"
              />
              {#if errors.PhoneNumber}
                <p class="mt-1 text-sm text-red-600">{errors.PhoneNumber}</p>
              {/if}
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="text"
                bind:value={formData.Pin}
                class="w-full px-3 py-2 border-2 border-[#02066F] rounded-lg bg-gray-300 font-bold focus:outline-none"
                disabled
                placeholder="Instructor Pin"
                readonly
              />
            </div>
            <div class="flex justify-center mt-6">
              <button
                type="submit"
                class="px-6 py-2 bg-[#02066F] text-white rounded-lg hover:bg-[#02066F]/80 transition-colors cursor-pointer"
              >
                {isEditing ? "Update" : "Submit"}
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
      on:click={() => (showAdminModal = false)}
    >
      <div
        class="bg-white rounded-lg w-full max-w-xs mx-4"
        on:click|stopPropagation
        transition:fade
      >
        <div
          class="flex w-full bg-[#02066F] justify-between p-3 pl-4 pr-4 items-center rounded-t-md text-center"
        >
          <h3 class="text-xl font-bold text-white">Admin Details</h3>
          <button
            class="text-gray-400 hover:text-white cursor-pointer text-5xl"
            on:click={() => (showAdminModal = false)}
          >
            &times;
          </button>
        </div>
        <div class="p-6">
          <!-- Device Info Display -->
          {#if selectedDevice}
            <div class="mb-4 p-2 bg-blue-50 border border-blue-200 rounded-md">
              <p class="text-sm text-blue-800">
                <strong>Device:</strong>
                {selectedDevice.DeviceName}
              </p>
            </div>
          {/if}

          <form on:submit|preventDefault={submitForm}>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="text"
                bind:value={formData.FName}
                class={`w-full px-3 py-2 font-bold border-2 ${errors.FName ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none`}
                placeholder="First Name"
              />
              {#if errors.FName}
                <p class="mt-1 text-sm text-red-600">{errors.FName}</p>
              {/if}
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="text"
                bind:value={formData.LName}
                class={`w-full px-3 py-2 font-bold border-2 ${errors.LName ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none`}
                placeholder="Last Name"
              />
              {#if errors.LName}
                <p class="mt-1 text-sm text-red-600">{errors.LName}</p>
              {/if}
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="text"
                bind:value={formData.PhoneNumber}
                on:input={handlePhoneInput}
                maxlength="14"
                class={`w-full px-3 py-2 font-bold border-2 ${errors.PhoneNumber ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none`}
                placeholder="Phone Number"
              />
              {#if errors.PhoneNumber}
                <p class="mt-1 text-sm text-red-600">{errors.PhoneNumber}</p>
              {/if}
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="text"
                bind:value={formData.Pin}
                class="w-full px-3 py-2 font-bold border-2 border-[#02066F] rounded-lg bg-gray-200 focus:outline-none"
                disabled
                placeholder="Pin"
                readonly
              />
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700 mb-1"
              ></label>
              <input
                type="email"
                bind:value={formData.Email}
                class={`w-full px-3 py-2 font-bold border-2 ${errors.Email ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none`}
                placeholder="Email"
              />
              {#if errors.Email}
                <p class="mt-1 text-sm text-red-600">{errors.Email}</p>
              {/if}
            </div>
            <div class="flex justify-center mt-6">
              <button
                type="submit"
                class="px-6 py-2 bg-[#02066F] text-white rounded-lg hover:bg-[#02066F]/90 transition-colors cursor-pointer"
              >
                {isEditing ? "Update" : "Submit"}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  {/if}

  <!-- SuperAdmin Modal -->
  {#if showSuperAdminModal}
    <div
      class="fixed inset-0 flex items-center justify-center z-50"
      style="background: rgba(0, 0, 0, 0.5)"
      on:click={() => (showSuperAdminModal = false)}
    >
      <div
        class="bg-white rounded-lg w-full max-w-xs mx-4"
        on:click|stopPropagation
        transition:fade
      >
        <div
          class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center rounded-t-md text-center"
        >
          <h3 class="text-xl font-bold p-2 text-white">SuperAdmin Details</h3>
          <button
            class="text-gray-400 hover:text-white cursor-pointer text-5xl"
            on:click={() => (showSuperAdminModal = false)}
          >
            &times;
          </button>
        </div>
        <div class="p-6">
          <!-- Device Info Display -->
          {#if selectedDevice}
            <div class="mb-4 p-2 bg-blue-50 border border-blue-200 rounded-md">
              <p class="text-sm text-blue-800">
                <strong>Device:</strong>
                {selectedDevice.DeviceName}
              </p>
            </div>
          {/if}

          <form on:submit|preventDefault={submitForm}>
            <div class="mb-4">
              <input
                id="superAdminFirstName"
                type="text"
                placeholder="First Name"
                bind:value={formData.FName}
                class={`w-full px-3 py-2 font-bold border-2 ${errors.FName ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none`}
              />
              {#if errors.FName}
                <p class="text-red-500 text-xs italic mt-1">{errors.FName}</p>
              {/if}
            </div>
            <div class="mb-4">
              <input
                id="superAdminLastName"
                type="text"
                placeholder="Last Name"
                bind:value={formData.LName}
                class={`w-full px-3 py-2 font-bold border-2 ${errors.LName ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none`}
              />
              {#if errors.LName}
                <p class="text-red-500 text-xs italic mt-1">{errors.LName}</p>
              {/if}
            </div>
            <div class="mb-4">
              <input
                id="superAdminPhone"
                type="text"
                bind:value={formData.PhoneNumber}
                on:input={handlePhoneInput}
                maxlength="14"
                class={`w-full px-3 py-2 font-bold border-2 ${errors.PhoneNumber ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none`}
                placeholder="Phone Number"
              />
              {#if errors.PhoneNumber}
                <p class="text-red-500 text-xs italic mt-1">
                  {errors.PhoneNumber}
                </p>
              {/if}
            </div>
            <div class="mb-4">
              <input
                id="superAdminPin"
                type="text"
                placeholder="Pin"
                bind:value={formData.Pin}
                class="w-full px-3 py-2 font-bold border-2 border-[#02066F] rounded-lg bg-gray-200 focus:outline-none"
                disabled
                readonly
              />
            </div>
            <div class="mb-4">
              <input
                id="superAdminEmail"
                type="email"
                placeholder="Email"
                bind:value={formData.Email}
                class={`w-full px-3 py-2 font-bold border-2 ${errors.Email ? "border-red-500" : "border-[#02066F]"} rounded-lg focus:outline-none`}
              />
              {#if errors.Email}
                <p class="text-red-500 text-xs italic mt-1">{errors.Email}</p>
              {/if}
            </div>
            <div class="flex justify-center mt-6">
              <button
                type="submit"
                class="px-4 py-2 bg-[#02066F] text-white rounded-md cursor-pointer hover:opacity-80"
              >
                {isEditing ? "Update" : "Submit"}
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
      on:click={() => (showDeleteModal = false)}
      on:keydown={(e) => {
        if (e.key === "Escape") showDeleteModal = false;
      }}
    >
      <div
        class="bg-white rounded-lg w-full max-w-md mx-4"
        on:click|stopPropagation
        transition:fade
      >
        <div
          class="flex w-full bg-[#02066F] justify-between p-2 pl-4 pr-4 items-center rounded-t-md text-center"
        >
          <h3 class="text-xl font-bold text-center text-white p-2">Delete</h3>
          <button
            class="text-gray-400 hover:text-white text-4xl cursor-pointer p-2"
            on:click={() => (showDeleteModal = false)}
          >
            ×
          </button>
        </div>
        <div class="py-6 px-0">
          {#if currentEmployee}
            <p class="text-center text-gray-800 font-bold text-lg mb-6">
              Are you sure, you want to remove the {currentEmployee.IsAdmin ===
              0
                ? "employee"
                : currentEmployee.IsAdmin === 1
                  ? "Admin"
                  : "SuperAdmin"}?
            </p>
          {:else}
            <p class="text-center font-bold mb-6">
              Are you sure, you want to remove the employee?
            </p>
          {/if}
          <div class="flex justify-center space-x-4">
            <button
              type="button"
              on:click={deleteEmployee}
              class="px-6 py-2 bg-[#02066F] opacity-80 hover:opacity-60 text-white rounded-md cursor-pointer"
            >
              Yes
            </button>
            <button
              type="button"
              on:click={() => (showDeleteModal = false)}
              class="px-6 py-2 border border-[#02066F] text-[#02066F] rounded-md hover:bg-gray-100 cursor-pointer"
            >
              No
            </button>
          </div>
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
  /* Import Font Awesome for icons */
  @import url("https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css");

  /* Custom styles for the table */
  table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
  }

  th,
  td {
    padding: 12px 15px;
    text-align: left;
    border-bottom: 1px solid #e5e7eb;
    border-right: 1px solid #d3d3d3;
    border-left: 1px solid #d3d3d3;
    text-align: center;
  }

  th {
    padding: 15px;
    background-color: #02066f;
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
