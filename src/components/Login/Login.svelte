<script>
    import { onMount } from 'svelte';
  
    let username = '';
    let password = '';
    let errorMsg = '';
    let isLoading = false;
  
    const key1 = new Uint8Array([16, 147, 220, 113, 166, 142, 22, 93, 241, 91, 13, 252, 112, 122, 119, 95]);
  
    const handleLogin = async () => {
      errorMsg = '';
      if (!username || !password) {
        errorMsg = "Please enter both username and password";
        return;
      }
      isLoading = true;
      try {
        const isAuthenticated = await loginCheck(username, password);
        if (isAuthenticated) {
          const cid1 = localStorage.getItem("companyID");
          await Promise.all([getCustmData(cid1), getTimeZone(cid1)]);
          window.location.href = 'employee_list.html';
        } else {
          errorMsg = "Invalid username or password";
          isLoading = false;
        }
      } catch (error) {
        console.error('Login error:', error);
        errorMsg = "An error occurred during authentication";
        isLoading = false;
      }
    };
  
    async function loginCheck(username, password) {
      try {
        const response = await fetch(`https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/company/getuser/${username}`);
        if (!response.ok) throw new Error('Network response was not ok');
        const data = await response.json();
        const decryptPassword = await decrypt(data["Password"], key1);
        const companyID = data["CID"];
  
        localStorage.setItem('companyID', companyID);
        localStorage.setItem('companyName', data["CName"]);
        localStorage.setItem('companyLogo', data["CLogo"]);
        localStorage.setItem('companyAddress', data["CAddress"]);
        localStorage.setItem('username', data["UserName"]);
        localStorage.setItem('password', data["Password"]);
        localStorage.setItem('reportType', data["ReportType"]);
  
        return data["UserName"] === username && decryptPassword === password;
      } catch (error) {
        console.error('Login check failed:', error);
        return false;
      }
    }
  
    async function decrypt(encryptedDataWithIV, key) {
      const buffer = new Uint8Array(atob(encryptedDataWithIV).split('').map(c => c.charCodeAt(0)));
      const iv = buffer.slice(0, 12);
      const encryptedData = buffer.slice(12);
      const algorithm = { name: 'AES-GCM', iv };
      const importedKey = await window.crypto.subtle.importKey('raw', key, algorithm, false, ['decrypt']);
      const decryptedData = await window.crypto.subtle.decrypt(algorithm, importedKey, encryptedData);
      return new TextDecoder().decode(decryptedData);
    }
  
    async function getCustmData(cid1) {
      try {
        const response = await fetch(`https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/customer/getUsingCID/${cid1}`);
        if (!response.ok) throw new Error(`Error: ${response.status}`);
        const data = await response.json();
        localStorage.setItem('customerID', data["CustomerID"]);
        localStorage.setItem('firstName', data["FName"]);
        localStorage.setItem('lastName', data["LName"]);
        localStorage.setItem('address', data["Address"]);
        localStorage.setItem('phone', data["PhoneNumber"]);
        localStorage.setItem('phoneNumber', data["PhoneNumber"]);
        localStorage.setItem('email', data["Email"]);
      } catch (error) {
        console.error('Error fetching customer data:', error);
      }
    }
  
    async function getTimeZone(cid1) {
      try {
        const response = await fetch(`https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/device/getAll/${cid1}`);
        if (!response.ok) {
          localStorage.setItem("TimeZone", "PST");
          return;
        }
        const data = await response.json();
        if (data.error === "No devices found !" || data.length === 0) {
          localStorage.setItem("TimeZone", "PST");
        } else {
          data.forEach(element => {
            localStorage.setItem("TimeZone", element.TimeZone);
          });
        }
      } catch (error) {
        console.error('Error fetching timezone:', error);
        localStorage.setItem("TimeZone", "PST");
      }
    }
  </script>
  
  <div class="min-h-screen flex flex-col md:flex-row">
    <!-- Left Column -->
    <div class="hidden md:flex xl:w-1/2 md:w-1/2 bg-[#D9E9FB] flex-col justify-center items-center xl:p-6">
      <div class="w-full flex flex-col items-center text-center">
        <img src="/icode-logo.png" alt="icode-logo" class="w-44 xl:w-94 md:w-32 mb-12 pt-42" />
        <h3 class="text-2xl xl:text-3xl md:text-2xl font-semibold text-gray-800 mb-2">Employee Time Tracking</h3>
        <p class="text-gray-900">One tap solution for simplifying and streamlining employee time logging and reporting.</p>
      </div>
    </div>
  
    <!-- Right Column (Form) -->
    <div class="w-full md:w-1/2 flex justify-center items-center py-16 px-6 sm:px-8 md:px-8 md:pt-30 lg:px-20">
      <div class="w-full bg-white rounded-xl p-6 sm:p-8 text-center">
        <h2 class=" sm:text-3xl md:text-3xl text-2xl pt-10 font-semibold text-gray-800 mb-10">Login</h2>
  
        <div class="mb-6">
          <input
            type="text"
            bind:value={username}
            placeholder="User Name"
            class="w-full xl:px-6 xl:py-4 md:px-2 md:py-2 px-10 py-2 rounded-lg border-2 border-[#02066F] focus:ring-2 focus:ring-[#02066F] focus:border-transparent text-center font-bold text-base sm:text-lg"
            required
          />
        </div>
  
        <div class="mb-4">
          <input
            type="password"
            bind:value={password}
            placeholder="Password"
            class="w-full xl:px-6 xl:py-4 md:px-2 md:py-2 px-6 py-2 rounded-lg border-2 border-[#02066F] focus:ring-2 focus:ring-[#02066F] focus:border-transparent text-center font-bold text-base sm:text-lg"
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
          on:click={handleLogin}
          class="w-full bg-[#02066F] text-white text-lg sm:text-xl py-4 md:py-3 rounded-lg transition duration-300 mb-4"
          disabled={isLoading}
        >
          {isLoading ? 'Logging in...' : 'Submit'}
        </button>
  
        <div>
          <span class="text-base sm:text-xl font-bold">Don't have an account?</span>
          <a href="/register" class="ml-2 text-[#02066F] text-base sm:text-xl font-bold hover:underline">Signup</a>
        </div>
      </div>
    </div>
  
    {#if isLoading}
      <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
        <div class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-[#02066F]"></div>
      </div>
    {/if}
  </div>
