<script lang="ts">
  // @ts-nocheck
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation'; // Use this if you're using SvelteKit
  
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
  let fileInput: File | null = null;
  
  const isAlpha = /^[a-zA-Z\s]+$/;
  
  function validateCompanyName() {
    if (companyName.trim() === '') {
      errorCompanyName = 'Company Name is required';
      return false;
    } else if (!isAlpha.test(companyName)) {
      errorCompanyName = "Only use letters, don't use digits";
      return false;
    }
    errorCompanyName = '';
    return true;
  }
  
  function validateUsername() {
    if (username.trim() === '') {
      errorUsername = 'Username is required';
      return false;
    } else if (!isAlpha.test(username)) {
      errorUsername = "Only use letters, don't use digits";
      return false;
    }
    errorUsername = '';
    return true;
  }
  
  function validatePassword() {
    if (password.trim() === '') {
      errorPassword = 'Password is required';
      return false;
    } else if (password.length < 8) {
      errorPassword = 'Password must be at least 8 characters';
      return false;
    }
    errorPassword = '';
    return true;
  }
  
  function validateRequiredFields() {
    let required = [companyName, companyStreet, companyCity, companyState, companyZip, username, password];
    return required.every(val => val.trim() !== '');
  }
  
  function handleFileInputChange(event) {
    const files = event.target.files;
    if (files && files.length > 0) {
      fileInput = files[0];
      fileName = fileInput.name;
    }
  }
  
  function validateForm(event: Event) {
    event.preventDefault();
  
    overlayVisible = true;
  
    const isCompanyNameValid = validateCompanyName();
    const isUsernameValid = validateUsername();
    const isPasswordValid = validatePassword();
    const isRequiredFieldsValid = validateRequiredFields();
  
    if (isCompanyNameValid && isUsernameValid && isPasswordValid && isRequiredFieldsValid) {
      document.querySelector('.progress-bar')?.setAttribute('style', 'width: 50%');
  
      // Store address
      localStorage.setItem('companyName', companyName);
      localStorage.setItem('companyStreet', companyStreet);
      localStorage.setItem('companyCity', companyCity);
      localStorage.setItem('companyState', companyState);
      localStorage.setItem('companyZip', companyZip);
      localStorage.setItem('username', username);
      localStorage.setItem('password', password);
  
      
      
      // Handle logo upload
      if (fileInput) {
        const reader = new FileReader();
        reader.onloadend = () => {
          localStorage.setItem('companyLogo', reader.result as string);
          // Redirect
          setTimeout(() => {
            overlayVisible = false;
            // window.location.href = "signup2.html"; // for SPA
            goto('/register2'); // if using SvelteKit
          }, 100);
        };
        reader.readAsDataURL(fileInput);
      } else {
        // No logo, still proceed
        setTimeout(() => {
          overlayVisible = false;
          goto('/register2'); // Redirect to next step
        }, 100);
      }
    } else {
      totalError = 'Please fix the errors';
      overlayVisible = false;
    }
  }
  
  // Sidebar toggle
  let sidebarOpen = false;
  
  function toggleSidebar() {
    sidebarOpen = !sidebarOpen;
  }
  
  function closeSidebarOnOutsideClick(event) {
    const sidebar = document.getElementById('sidebar');
    const toggler = document.querySelector('.navbar-toggler');
  
    if (
      sidebar &&
      !sidebar.contains(event.target as Node) &&
      !(toggler && toggler.contains(event.target as Node))
    ) {
      sidebarOpen = false;
    }
  }
  
  onMount(() => {
    document.addEventListener('click', closeSidebarOnOutsideClick);
  });
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
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold focus:outline-none"
                required
              />
              <input
                type="text"
                bind:value={companyCity}
                placeholder="Company City"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold focus:outline-none"
                required
              />
            </div>
            <div class="flex flex-col md:flex-row gap-4 mt-4 mb-4">
              <input
                type="text"
                bind:value={companyState}
                placeholder="Company State"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold focus:outline-none"
                required
              />
              <input
                type="text"
                bind:value={companyZip}
                placeholder="Company Zip"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold focus:outline-none"
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
            on:click={validateForm}
            class="w-full bg-[#02066F] text-white py-3 rounded-lg text-lg mt-4 cursor-pointer"
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