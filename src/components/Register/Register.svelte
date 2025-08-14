<script lang="ts">
  // @ts-nocheck
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation'; // Use this if you're using SvelteKit
  import { readonly } from 'svelte/store';
  
  let isFreeTrail = false;
  let companyName = '';
  let companyLogo = '';
  let companyStreet = '';
  let companyCity = '';
  let companyState = '';
  let companyZip = '';
  let NoOfDevices = '';
   
  let NoOfEmployees = '';
  
  //  let employeeCount = isFreeTrail ? 10 : NoOfEmployees;

  onMount(() => {
    isFreeTrail = localStorage.getItem('trial') === 'true';
    console.log("isFreeTrail", isFreeTrail);
    NoOfDevices = isFreeTrail ? 1 : '';
    NoOfEmployees = isFreeTrail ? 10 : '';
  });
  
  let errorCompanyName = '';
  let totalError = '';
  let overlayVisible = false;
  let fileName = '';
  let fileInput: File | null = null;

 
  
  
  const isAlpha = /^[a-zA-Z\s]+$/;
  
  // Validation functions
  function validateCompanyName() {
    if (companyName.trim() === '') {
      errorCompanyName = '';
      return false;
    } else if (!isAlpha.test(companyName)) {
      errorCompanyName = "Only use letters, don't use digits";
      return false;
    }
    errorCompanyName = '';
    return true;
  }
  
  function validateRequiredFields() {
  let requiredStrings = [companyName, companyStreet, companyCity, companyState, companyZip];
  let requiredNumbers = [NoOfDevices, NoOfEmployees];

  const areStringsValid = requiredStrings.every(val => typeof val === 'string' && val.trim() !== '');
  const areNumbersValid = requiredNumbers.every(val => typeof val === 'number' && !isNaN(val));

  return areStringsValid && areNumbersValid;
}
  
  function handleFileInputChange(event) {
    const files = event.target.files;
    if (files && files.length > 0) {
      fileInput = files[0];
      fileName = fileInput.name;
      // Store the file for later use
      companyLogo = URL.createObjectURL(fileInput);
    }
  }
  
  function validateForm(event: Event) {
    event.preventDefault();
  
    overlayVisible = true;
    totalError = '';
  
    const isCompanyNameValid = validateCompanyName();
    // const isUsernameValid = validateUsername();
    // const isPasswordValid = validatePassword();
    const isRequiredFieldsValid = validateRequiredFields();
  
    if (isCompanyNameValid && isRequiredFieldsValid) {
      // Update progress bar
      const progressBar = document.querySelector('.progress-bar');
      if (progressBar) {
        progressBar.style.width = '50%';
      }
  
      // Store address
      localStorage.setItem('companyName', companyName);
      localStorage.setItem('companyStreet', companyStreet);
      localStorage.setItem('companyCity', companyCity);
      localStorage.setItem('companyState', companyState);
      localStorage.setItem('companyZip', companyZip);
      localStorage.setItem('noOfDevices', NoOfDevices);
      localStorage.setItem('noOfEmployees', NoOfEmployees);
  
      // Handle logo upload
      if (fileInput) {
        const reader = new FileReader();
        reader.onloadend = () => {
          localStorage.setItem('companyLogo', reader.result as string);
          // Redirect
          setTimeout(() => {
            overlayVisible = false;
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
</script>
  
  
  <div class="flex flex-col md:flex-row min-h-screen">
    <!-- Left Section -->
    <div class="hidden md:flex md:w-1/2 bg-blue-100 flex-col justify-center items-center p-5  ">
      <!-- <img src="/icode-logo.png" alt="icode-logo" class="w-18 mb-4" /> -->
       <img src="/tap-time-logo.png" alt="icode-logo" class="w-44 xl:w-94 md:w-32 md:pt-2 xl:pt-8 ">
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
          <div class="fixed inset-0 flex justify-center items-center z-50"
          style="background: rgba(0, 0, 0, 0.5)">
            <div class="border-4 border-t-4 border-[#02066F] rounded-full w-10 h-10 animate-spin"></div>
          </div>
        {/if}
      
        <form class="w-full pt-14 md:pt-12 ">
          <h2 class="text-center text-3xl xl:text-3xl md:text-2xl text-gray-800 font-semibold mb-4">Signup</h2>
      
          <!-- Horizontal line -->
          <hr class="w-full h-4 bg-gray-200 rounded-full border-4 mb-6 border-gray-200"  />

      
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
          
          <h1 class="text-[#02066F] text-md md:text-lg font-bold text-center mb-2">Company Logo</h1>
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
                placeholder="Company Address Line 1"
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
              type="number"
              bind:value={NoOfDevices}
              placeholder={isFreeTrail ? '1' : "No Of Devices" }
              class="w-full outline-none font-bold p-2 xl:p-6 md:p-3"
              readonly={isFreeTrail} required={!isFreeTrail}
            />
          </div>
      
          <!-- Password -->
          <div class="border-2 border-[#02066F] rounded-lg mb-4">
            <input
              type="number"
              bind:value={NoOfEmployees}
              placeholder={isFreeTrail ? '10' : "No Of Employees"}
              class="w-full outline-none font-bold p-2 xl:p-6 md:p-3"
              readonly={isFreeTrail} required={!isFreeTrail}
            />
          </div>
      
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
  <div class="fixed inset-0 flex items-center justify-center z-50"
  style="background: rgba(0, 0, 0, 0.5)">
    <div class="text-white text-xl font-semibold">Processing...</div>
  </div>
{/if}
      </div>
      
  </div>