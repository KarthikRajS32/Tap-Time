<script>
// @ts-nocheck

    import { onMount } from 'svelte';
    let companyName = '';
    let companyLogo = '';
    let companyStreet = '';
    let companyCity = '';
    let companyState = '';
    let companyZip = '';
    let username = '';
    let password = '';
    let errorCompanyName = '';
    let errorUsername = '';
    let errorPassword = '';
    let totalError = '';
    let overlayVisible = false;
    let fileName = '';
  
    function validateCompanyName() {
      errorCompanyName = companyName.trim() === '' ? 'Company Name is required' : '';
    }
  
    function validateUsername() {
      errorUsername = username.trim() === '' ? 'Username is required' : '';
    }
  
    function validatePassword() {
      errorPassword = password.trim() === '' ? 'Password is required' : '';
    }

    
  
 
  </script>
  
  <div class="flex flex-col md:flex-row min-h-screen">
    <!-- Left Section -->
    <div class="hidden md:flex md:w-1/2 bg-blue-100 flex-col justify-center items-center p-5  ">
      <!-- <img src="/icode-logo.png" alt="icode-logo" class="w-18 mb-4" /> -->
       <img src="/icode-logo.png" alt="icode-logo" class="w-44 xl:w-94 md:w-32 md:pt-2 xl:pt-8 ">
      <div>
       <h3 class="text-center text-3xl xl:text-3xl md:text-2xl text-gray-800 font-semibold xl:pt-20 md:pt-18">Join Us Today</h3>
      <p class="text-center text-gray-700 mt-2 ">
        Create an account to unlock seamless time tracking and boost your team's efficiency.
      </p>
    </div>
    </div>
  
    <!-- Right Section -->
    
      <div class="w-full md:w-1/2 xl:w-1/2 flex justify-center items-center p-6 pt-10 xl:pt-16 md:pt-16 ">
        {#if overlayVisible}
          <div class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50">
            <div class="border-4 border-t-4 border-[#02066F] rounded-full w-10 h-10 animate-spin"></div>
          </div>
        {/if}
      
        <form class="w-full pt-16 md:pt-24 ">
          <h2 class="text-center text-3xl xl:text-3xl md:text-2xl text-gray-800 font-semibold mb-4">Signup</h2>
      
          <!-- Horizontal line -->
          <hr class="w-full rounded-full border-4 mb-6" style="border-color:  rgb(234,234,234);" />

      
          {#if totalError}
            <p class="text-red-600 text-sm mb-2">{totalError}</p>
          {/if}
      
          <!-- Company Name -->
          <div class="border-2 border-[#02066F] rounded-lg  mb-4">
            <input
              type="text"
              bind:value={companyName}
              on:blur={validateCompanyName}
              placeholder="Company Name"
              class="w-full outline-none font-bold p-3 xl:p-6 md:p-3"
              required
            />
          </div>
          {#if errorCompanyName}
            <p class="text-red-600 text-sm mb-2">{errorCompanyName}</p>
          {/if}
      
          <!-- Company Logo -->
          

          <div class="border-2 border-[#02066F] rounded-lg mb-4 p-4 ">
            <label
              for="companyLogo"
              class="inline-block font-bold xl:text-lg md:text-lg text-base py-1 px-2 border-2 rounded cursor-pointer transition " style="border-color:  rgb(234,234,234); background-color: #f0f0f0; border: 1px solid gray ;">
              Choose File
            </label>
            <span class=" xl:text-lg md:text-lg text-gray-800 text-base font-bold">{fileName || 'No file chosen'}</span>
            <input
              id="companyLogo"
              type="file"
              accept="image/png, image/jpeg, image/jpg"
              class="hidden"
              on:change={(e) => fileName = e.target.files[0]?.name}
            />
          </div>
          
      
          <!-- Company Address -->
          <div class="border-2 border-[#02066F] rounded-lg p-2 xl:p-4 md:p-4 xl:p-4 md:p-4 mb-4">
            <p class="text-center text-[#02066F] font-bold mb-4">Company Address</p>
            <div class="flex flex-col md:flex-row gap-4">
              <input
                type="text"
                bind:value={companyStreet}
                placeholder="Company Street"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold"
                required
              />
              <input
                type="text"
                bind:value={companyCity}
                placeholder="Company City"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold"
                required
              />
            </div>
            <div class="flex flex-col md:flex-row gap-4 mt-4 mb-4">
              <input
                type="text"
                bind:value={companyState}
                placeholder="Company State"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold"
                required
              />
              <input
                type="text"
                bind:value={companyZip}
                placeholder="Company Zip"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold"
                required
              />
            </div>
          </div>
      
          <!-- Username -->
          <div class="border-2 border-[#02066F] rounded-lg mb-4">
            <input
              type="text"
              bind:value={username}
              on:blur={validateUsername}
              placeholder="Username"
              class="w-full outline-none font-bold p-2 xl:p-6 md:p-3"
              required
            />
          </div>
          {#if errorUsername}
            <p class="text-red-600 text-sm mb-2">{errorUsername}</p>
          {/if}
      
          <!-- Password -->
          <div class="border-2 border-[#02066F] rounded-lg mb-4">
            <input
              type="password"
              bind:value={password}
              on:blur={validatePassword}
              placeholder="Password"
              class="w-full outline-none font-bold p-2 xl:p-6 md:p-3"
              required
            />
          </div>
          {#if errorPassword}
            <p class="text-red-600 text-sm mb-2">{errorPassword}</p>
          {/if}
      
          <!-- Submit Button -->
          <button
            type="submit"
            class="w-full bg-[#02066F] text-white py-3 rounded-lg text-lg mt-4 hover:opacity-90"
          >
            Next
          </button>
        </form>
        {#if overlayVisible}
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="text-white text-xl font-semibold">Processing...</div>
  </div>
{/if}
      </div>
      
  </div>