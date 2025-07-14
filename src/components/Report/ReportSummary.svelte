<script lang="ts">
    //@ts-nocheck
    import { onMount, afterUpdate } from 'svelte';
    import flatpickr from 'flatpickr';
    import 'flatpickr/dist/flatpickr.min.css';
    import DataTable from 'datatables.net-dt';
  
    // Types
    type Employee = { FName: string; EmpID: string; [key: string]: any; };
    type TableRow = {
      EmpID?: string; CID?: string; Date?: string; TypeID?: string;
      CheckInTime?: string; CheckOutTime?: string; CheckInSnap?: string;
      CheckOutSnap?: string; checkInTimeFormatted?: string; needsCheckout?: boolean;
      checkoutTime?: string; empty?: boolean; message?: string;
    };
  
    // State
    let employeeDetails: Record<string, Employee> = {};
    let tableData: TableRow[] = [];
    let dataTable: any = null;
    let tableElement: HTMLTableElement;
    let showLogoutModal = false;
    let showHomeModal = false;
  
    let currentDate = '';
    let isLoading = false;
    let showModal = false;
  
    let formData = {
      employee: '',
      type: '',
      date: '',
      checkinTime: '',
      checkoutTime: ''
    };
  
    let errors = {
      employee: '',
      type: '',
      date: '',
      checkinTime: '',
      checkoutTime: ''
    };
  
    // Constants
    const cid = localStorage.getItem('companyID') || '';
    if (!cid) {
      alert('Company ID missing. Please login again.');
      window.location.href = '/login';
    }
  
    const BASE = 'https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test';
  
    onMount(async () => {
      const today = new Date().toISOString().substring(0, 10);
      currentDate = today;
      formData.date = today;
  
      flatpickr('#date-picker', {
        dateFormat: 'Y-m-d',
        defaultDate: today,
        onChange: (dates: Date[]) => {
          formData.date = dates[0].toISOString().substring(0, 10);
        }
      });
  
      await fetchEmployeeData();
      await viewCurrentDateReport();
    });
    
    const convertToAmPm = formatToAmPm;
  
    afterUpdate(() => {
      if (dataTable) dataTable.destroy();
      if (tableElement) {
        dataTable = new DataTable(tableElement, {
          date: true,
          paging: fasle,
          searching: false,
          info: false,
          ordering: false
        });
      }
    });
  
    // Fetch employees
    async function fetchEmployeeData() {
      try {
        const res = await fetch(`${BASE}/employee/getall/${cid}`);
        if (!res.ok) throw new Error('Error fetching employees');
        const arr = await res.json();
        arr.forEach((e: Employee) => {
          employeeDetails[e.EmpID] = e;
        });
      } catch (err) {
        console.error(err);
      }
    }
  
    // View report
    async function viewCurrentDateReport() {
      isLoading = true;
      try {
        const res = await fetch(`${BASE}/dailyreport/getdatebasedata/${cid}/${formData.date}`);
        if (!res.ok) throw new Error('Error fetching report data');
        const arr = await res.json();
        if (!arr.length) {
          tableData = [];
          return;
        }
        tableData = arr.map((row: any) => ({
          ...row,
          checkInTimeFormatted: formatToAmPm(new Date(row.CheckInTime)),
          needsCheckout: !row.CheckOutTime,
          checkoutTime: ''
        }));
      } catch (err) {
        console.error(err);
      } finally {
        isLoading = false;
      }
    }
  
    function formatToAmPm(date: Date) {
      let h = date.getHours();
      const m = date.getMinutes();
      const ampm = h >= 12 ? 'PM' : 'AM';
      h = h % 12 || 12;
      return `${h}:${String(m).padStart(2, '0')} ${ampm}`;
    }
  
    function parseDateTime(date: string, time: string) {
      return `${date}T${time.padStart(2, '0')}:00`;
    }
  
    // Checkout logic
    async function handleCheckout(row: TableRow) {
      if (!row.checkoutTime) return;
      const checkIn = row.CheckInTime!;
      const checkOut = parseDateTime(row.Date!, row.checkoutTime);
      const diff = new Date(checkOut).getTime() - new Date(checkIn).getTime();
      if (diff <= 0) return alert('Checkout must be after checkin');
  
      const minutes = diff / 1000 / 60;
      const timeWorked = `${String(Math.floor(minutes / 60)).padStart(2, '0')}:${String(minutes % 60).padStart(2, '0')}`;
  
      try {
        const res = await fetch(`${BASE}/dailyreport/update/${row.EmpID}/${cid}/${encodeURIComponent(checkIn)}`, {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            CID: cid,
            EmpID: row.EmpID,
            Date: row.Date,
            TypeID: row.TypeID,
            CheckInTime: checkIn,
            CheckOutTime: checkOut,
            TimeWorked: timeWorked,
            CheckInSnap: row.CheckInSnap,
            CheckOutSnap: row.CheckOutSnap,
            LastModifiedBy: 'Admin'
          })
        });
        if (!res.ok) throw await res.text();
        await viewCurrentDateReport();
      } catch (e) {
        console.error('Checkout error:', e);
      }
    }
  
    // Validation
    function validateForm() {
      errors = { employee: '', type: '', date: '', checkinTime: '', checkoutTime: '' };
      if (!formData.employee) errors.employee = 'Select employee.';
      if (!formData.type) errors.type = 'Select type.';
      if (!formData.checkinTime) errors.checkinTime = 'Select check-in.';
      if (!formData.checkoutTime || formData.checkoutTime <= formData.checkinTime)
        errors.checkoutTime = 'Checkout after checkin.';
      return !Object.values(errors).some(e => e);
    }
  
    // Submit new entry
    async function submitForm() {
      if (!validateForm()) return;
      const checkIn = parseDateTime(formData.date, formData.checkinTime);
      const checkOut = parseDateTime(formData.date, formData.checkoutTime);
      const diff = new Date(checkOut).getTime() - new Date(checkIn).getTime();
      if (diff <= 0) return alert('Checkout must be after checkin');
  
      const minutes = diff / 1000 / 60;
      const timeWorked = `${String(Math.floor(minutes / 60)).padStart(2, '0')}:${String(minutes % 60).padStart(2, '0')}`;
  
      try {
        const res = await fetch(`${BASE}/dailyreport/create`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            CID: cid,
            EmpID: formData.employee,
            Date: formData.date,
            TypeID: formData.type,
            CheckInTime: checkIn,
            CheckOutTime: checkOut,
            TimeWorked: timeWorked,
            CheckInSnap: null,
            CheckOutSnap: null,
            LastModifiedBy: 'Admin'
          })
        });
        if (!res.ok) throw await res.text();
        formData = { employee: '', type: '', date: currentDate, checkinTime: '', checkoutTime: '' };
        showModal = false;
        await viewCurrentDateReport();
      } catch (e) {
        console.error('Submit error:', e);
      }
    }
     let reportTypeHeading: string = '';
    
    onMount(() => {
        const selectedValue = localStorage.getItem('reportType');
        reportTypeHeading = `${selectedValue} Report`;
        // Rest of your onMount code...
    });
  </script>


  <div class="pt-16 md:pt-18 sm:pt-2">

    {#if isLoading}
        <div class="fixed inset-0 flex items-center justify-center z-50"    
        style="background: rgba(0, 0, 0, 0.5)">
        <div class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"></div>
        </div>
    {/if}
   
  
    <!-- Navigation -->
    <nav class="bg-white shadow">
      <div class="max-w-8xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex flex-col md:flex-row justify-between items-center h-auto md:h-16 py-4 md:py-0 justify-end">
          
          <!-- Menu Links - visible on all devices -->
          <div class="flex flex-col md:flex-row space-y-2 md:space-y-0 md:space-x-4 text-center w-auto md:w-auto">
            <a href="/reportsummary" class="px-4 py-2 bg-[#02066F] text-white font-semibold rounded-full">Today's Report</a>
            <a href="/daywisereport" class="px-4 py-2 text-[#02066F] font-semibold rounded-full">Day Wise Report</a>
            <a href="/salariedreport" class="px-4 py-2 text-[#02066F] font-semibold rounded-full">{reportTypeHeading}</a>
          </div>
    
        </div>
      </div>
    </nav>
    
      
  
    <!-- Main Content -->
    <main class="bg-gray-100 px-4 sm:px-6 lg:px-8 py-6">
        <div>
          <!-- Title -->
          <div class="text-center mb-4 pt-6">
            <h2 class="text-base sm:text-md md:text-xl font-bold text-gray-800">Current Day Report</h2>
          </div>
      
          <!-- Date & Add Entry Button -->
          <div class="max-w-5xl mx-auto flex flex-row md:flex-row justify-between items-center mb-6 gap-4">
            <h3 class="text-base sm:text-lg font-semibold text-center md:text-left">Date: {currentDate}</h3>
            <button 
              class="bg-white border border-blue-900 text-blue-900 px-2 md:px-6 py-2 rounded-md font-semibold cursor-pointer transition"
              on:click={() => showModal = true}
            >
              Add Entry
            </button>
          </div>
        </div>
      
        <!-- Table Card -->
        <div class="max-w-5xl mx-auto bg-white rounded-xl shadow-md overflow-hidden mb-8 border border-gray-300">
          <div class="p-4 sm:p-6 overflow-x-auto">
            <table bind:this={tableElement} class="min-w-full border border-gray-300 text-sm">
              <thead class="bg-[#02066F] text-white">
                <tr>
                  <th class="text-base px-4 sm:px-6 py-3 text-center font-bold border-r">Employee ID</th>
                  <th class="text-base px-4 sm:px-6 py-3 text-center font-bold border-r">Name</th>
                  <th class="text-base px-4 sm:px-6 py-3 text-center font-bold border-r">Check-in Time</th>
                  <th class="text-base px-4 sm:px-6 py-3 text-center font-bold border-r">Check-out Time</th>
                  <th class="text-base px-4 sm:px-6 py-3 text-center font-bold border-r">Action</th>
                </tr>
              </thead>
      
              <tbody class="bg-white divide-y divide-gray-200">
                {#if tableData.length === 0}
                  <tr>
                    <td colspan="5" class="px-6 py-4 text-center">No Records Found</td>
                  </tr>
                {:else if tableData[0]?.empty}
                  <tr>
                    <td colspan="5" class="px-6 py-4 text-center">{tableData[0].message}</td>
                  </tr>
                {:else}
                  {#each tableData as row (row.Pin || '')}
                    <tr class="hover:bg-gray-50">
                      <td class="px-6 py-4 text-center">{row.Pin}</td>
                      <td class="px-6 py-4 text-center">{row.Name?.split(" ")[0]}</td>
                      <td class="px-6 py-4 text-center">{row.checkInTimeFormatted}</td>
      
                      {#if row.needsCheckout}
                        <td class="px-6 py-4 text-center">
                          <input 
                            type="time" 
                            class="border border-gray-300 rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
                            bind:value={row.checkoutTime}
                          />
                        </td>
                        <td class="px-6 py-4 text-center">
                          <button 
                            class="bg-blue-900 text-white px-4 py-2 rounded disabled:bg-gray-300 disabled:cursor-not-allowed"
                            disabled={!row.checkoutTime}
                            on:click={() => row.checkoutTime && handleCheckout(row, row.checkoutTime)}
                          >
                            Check-out
                          </button>
                        </td>
                      {:else}
                        <td class="px-6 py-4 text-center">{row.CheckOutTime ? convertToAmPm(new Date(row.CheckOutTime)) : ''}</td>
                        <td class="px-6 py-4 text-center">
                          <button class="bg-gray-300 text-gray-600 px-4 py-2 rounded cursor-not-allowed" disabled>
                            Check-out
                          </button>
                        </td>
                      {/if}
                    </tr>
                  {/each}
                {/if}
              </tbody>
            </table>
          </div>
        </div>
      </main>
      
  
    <!-- Modals -->
    {#if showModal}
      <div class="fixed inset-0  flex items-center justify-center z-50 p-4"
      style="background: rgba(0, 0, 0, 0.5)">
        <div class="bg-white rounded-lg shadow-xl w-full max-w-xs">
          <div class="bg-[#02066F] text-white px-6 py-4 rounded-t-lg flex justify-between items-center">
            <h3 class="text-lg font-bold text-center">Add entry</h3>
            <button on:click={() => showModal = false} class="text-gray-400 text-4xl hover:text-white cursor-pointer">
              &times;
            </button>
          </div>
          <div class="p-6">
            <form on:submit|preventDefault={submitForm}>
              <div class="mb-4">
                <select 
                  class="w-full border-2 text-[#02066F] border-[#02066F] rounded-lg px-4 py-2 font-bold focus:outline-none"
                  bind:value={formData.employee}
                >
                  <option value="">Select Employee</option>
                  <!-- {#each Object.keys(employeeDetails) as employee}
                    <option value={employee}>{employee}</option>
                  {/each} -->
                  <option value="Arjava">Arjava</option>
                  <option value="Mani">Mani</option>
                  <option value="balaji">balaji</option>
                  <option value="Test">Test</option>
                  <option value="PITCHAIMANI">PITCHAIMANI</option>
                  <option value="Balaji">Balaji</option>
                </select>
                {#if errors.employee}
                  <p class="text-red-500 text-sm mt-1 text-center">{errors.employee}</p>
                {/if}
              </div>
  
              <div class="mb-4">
                <select 
                  class="w-full border-2 border-[#02066F] text-[#02066F] rounded-lg px-4 py-2 font-bold focus:outline-none "
                  bind:value={formData.type}
                >
                  <option value="">Select Type</option>
                  <option value="Belt">Belt</option>
                  <option value="Path">Path</option>
                  <option value="Camp">Camp</option>
                  <option value="External">External</option>
                  <option value="Trial">Trial</option>
                  <option value="Reception">Reception</option>
                </select>
                {#if errors.type}
                  <p class="text-red-500 text-sm mt-1 text-center">{errors.type}</p>
                {/if}
              </div>
  
              <div class="mb-4">
                <input 
                  type="date" 
                  class="w-full border-2 border-[#02066F] text-[#02066F] rounded-lg px-4 py-2 font-bold focus:outline-none "
                  bind:value={formData.date}
                  max={currentDate}
                />
                {#if errors.date}
                  <p class="text-red-500 text-sm mt-1 text-center">{errors.date}</p>
                {/if}
              </div>
  
              <div class="mb-4">
                <input 
                  type="time" 
                  class="w-full border-2 border-[#02066F] text-[#02066F] rounded-lg px-4 py-2 font-bold focus:outline-none "
                  bind:value={formData.checkinTime}
                  on:change={() => formData.checkoutTime = ''}
                />
                {#if errors.checkinTime}
                  <p class="text-red-500 text-sm mt-1 text-center">{errors.checkinTime}</p>
                {/if}
              </div>
  
              <div class="mb-6">
                <input 
                  type="time" 
                  class="w-full border-2 border-[#02066F] text-[#02066F] rounded-lg px-4 py-2 font-bold focus:outline-none"
                  bind:value={formData.checkoutTime}
                  disabled={!formData.checkinTime}
                />
                {#if errors.checkoutTime}
                  <p class="text-red-500 text-sm mt-1 text-center">{errors.checkoutTime}</p>
                {/if}
              </div>
  
              <div class="text-center">
                <button 
                  type="submit" 
                  class="bg-[#02066F] text-white px-8 py-2 rounded-lg font-bold cursor-pointer transition"
                >
                  Add
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    {/if}  
    
  </div>