<script lang="ts">
  import { onMount } from 'svelte';

  let username = '';
  let password = '';
  let errorMsg = '';
  let loading = false;
  let sidebarOpen = false;
  let logoSrc = '';

  // AES-GCM key used for decryption
  const key1 = new Uint8Array([16, 147, 220, 113, 166, 142, 22, 93, 241, 91, 13, 252, 112, 122, 119, 95]);

  onMount(() => {
    // Retrieve the logo from local storage
    const storedLogo = localStorage.getItem('companyLogo');
    if (storedLogo) {
      logoSrc = storedLogo;
    }

    // Click outside sidebar handler
    const handleClickOutside = (event: MouseEvent) => {
      const sidebar = document.getElementById('sidebar');
      const toggler = document.querySelector('.navbar-toggler');
      
      if (sidebar && toggler) {
        const isClickInside = sidebar.contains(event.target as Node) || 
                             toggler.contains(event.target as Node);
        if (!isClickInside) {
          sidebarOpen = false;
        }
      }
    };

    document.addEventListener('click', handleClickOutside);

    return () => {
      document.removeEventListener('click', handleClickOutside);
    };
  });

  function toggleSidebar() {
    sidebarOpen = !sidebarOpen;
  }

  async function decrypt(encryptedDataWithIV: string, key: Uint8Array): Promise<string> {
    const buffer = new Uint8Array(atob(encryptedDataWithIV).split('').map(char => char.charCodeAt(0)));
    const iv = buffer.slice(0, 12);
    const encryptedData = buffer.slice(12);

    const algorithm = { name: 'AES-GCM', iv };
    const cryptoKey = await crypto.subtle.importKey('raw', key, algorithm, false, ['decrypt']);

    const decryptedData = await crypto.subtle.decrypt(algorithm, cryptoKey, encryptedData);
    return new TextDecoder().decode(decryptedData);
  }

  async function getCustomerData(cid: string) {
    const url = `https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/customer/getUsingCID/${cid}`;
    try {
      const response = await fetch(url);
      const data = await response.json();
      localStorage.setItem('customerID', data.CustomerID);
      localStorage.setItem('firstName', data.FName);
      localStorage.setItem('lastName', data.LName);
      localStorage.setItem('address', data.Address);
      localStorage.setItem('phone', data.PhoneNumber);
      localStorage.setItem('phoneNumber', data.PhoneNumber);
      localStorage.setItem('email', data.Email);
    } catch (err) {
      console.error("Error fetching customer data:", err);
    }
  }

  async function getTimeZone(cid: string) {
    const url = `https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/device/getAll/${cid}`;
    try {
      const res = await fetch(url);
      const data = await res.json();
      if (!data.length || data.error === 'No devices found !') {
        localStorage.setItem('TimeZone', 'PST');
      } else {
        // Use the first device's timezone or default to PST
        localStorage.setItem('TimeZone', data[0]?.TimeZone || 'PST');
      }
    } catch (err) {
      console.error("Error fetching timezone:", err);
      localStorage.setItem('TimeZone', 'PST');
    }
  }

  async function handleSubmit() {
    errorMsg = '';
    if (!username.trim() || !password.trim()) {
      errorMsg = 'Please enter both username and password';
      return;
    }

    loading = true;

    try {
      const res = await fetch(`https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/company/getuser/${username}`);
      if (!res.ok) throw new Error('User fetch failed');

      const data = await res.json();

      if (!data || !data.Password || !data.UserName) {
        errorMsg = 'Invalid response from server';
        return;
      }

      const decryptedPwd = await decrypt(data.Password, key1);

      const isAuthenticated = data.UserName === username && decryptedPwd === password;

      if (!isAuthenticated) {
        errorMsg = 'Invalid username or password';
        return;
      }

      // Set LocalStorage
      localStorage.setItem('companyID', data.CID);
      localStorage.setItem('companyName', data.CName);
      localStorage.setItem('companyLogo', data.CLogo);
      localStorage.setItem('companyAddress', data.CAddress);
      localStorage.setItem('username', data.UserName);
      localStorage.setItem('password', data.Password);
      localStorage.setItem('reportType', data.ReportType);

      // Update logo if it exists
      if (data.CLogo) {
        logoSrc = data.CLogo;
      }

      await Promise.all([
        getCustomerData(data.CID),
        getTimeZone(data.CID)
      ]);

      window.location.href = '/employeelist';
    } catch (err) {
      console.error(err);
      errorMsg = 'An error occurred during login';
    } finally {
      loading = false;
    }
  }
</script>

<!-- Page layout -->
<div class="min-h-screen flex flex-col md:flex-row">
  <!-- Left side image & intro -->
  <div class="hidden md:flex xl:w-1/2 md:w-1/2 bg-[#D9E9FB] flex-col justify-center items-center xl:p-6">
    <div class="w-full flex flex-col items-center text-center">
      <img src="/icode-logo.png" alt="icode-logo" class="w-44 xl:w-94 md:w-32 mb-12 pt-42" />
      <h3 class="text-2xl xl:text-3xl md:text-2xl font-semibold text-gray-800 mb-2">Employee Time Tracking</h3>
      <p class="text-gray-900">One tap solution for simplifying and streamlining employee time logging and reporting.</p>
    </div>
  </div>

  <!-- Right side login form -->
  <div class="w-full md:w-1/2 flex justify-center items-center py-16 px-6 sm:px-8 md:px-8 md:pt-30 lg:px-20">
    <div class="w-full bg-white rounded-xl p-6 sm:p-8 text-center">
      
      <!-- {#if companyLogo}
        <div class="mb-6">
          <img src={companyLogo} alt="Company Logo" class="w-24 h-auto mx-auto" />
        </div>
      {/if} -->

      <h2 class="sm:text-3xl md:text-3xl text-2xl pt-4 font-semibold text-gray-800 mb-10">Login</h2>

      <div class="mb-6">
        <input
          type="text"
          bind:value={username}
          placeholder="User Name"
          class="w-full xl:px-6 xl:py-4 md:px-2 md:py-2 px-10 py-2 rounded-lg border-2 border-[#02066F] focus:outline-none text-center font-bold text-base sm:text-lg"
          required
        />
      </div>

      <div class="mb-4">
        <input
          type="password"
          bind:value={password}
          placeholder="Password"
          class="w-full xl:px-6 xl:py-4 md:px-2 md:py-2 px-6 py-2 rounded-lg border-2 border-[#02066F] focus:outline-none text-center font-bold text-base sm:text-lg"
          required
        />
      </div>

      {#if errorMsg}
        <div class="text-red-500 font-semibold mb-4">{errorMsg}</div>
      {/if}

      <div class="mb-6">
        <a href="/forgetpassword" class="text-[#02066F] font-bold">Forget password?</a>
      </div>

      <button
        on:click={handleSubmit}
        class="w-full bg-[#02066F] text-white text-lg cursor-pointer sm:text-xl py-4 md:py-3 rounded-lg transition duration-300 mb-4 disabled:opacity-50 disabled:cursor-not-allowed"
        disabled={loading}
      >
        {loading ? 'Logging in...' : 'Submit'}
      </button>

      <div>
        <span class="text-base sm:text-xl font-bold">Don't have an account?</span>
        <a href="/register" class="ml-2 text-[#02066F] text-base sm:text-xl font-bold hover:underline">Signup</a>
      </div>
    </div>
  </div>

  {#if loading}
    <div class="fixed inset-0 flex items-center justify-center z-50"
    style="background: rgba(0, 0, 0, 0.5)">
      <div class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-[#02066F]"></div>
    </div>
  {/if}
</div>
