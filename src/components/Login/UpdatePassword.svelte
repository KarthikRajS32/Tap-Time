<script lang="ts">
    import { onMount } from 'svelte';
  
    let newPassword = '';
    let confirmPassword = '';
    let errorMessage = '';
    
  
    const updatePassword = async () => {
      const CID = localStorage.getItem('companyID');
      const companyName = localStorage.getItem('companyName');
      const companyLogo = localStorage.getItem('companyLogo');
      const companyAddress = localStorage.getItem('companyAddress');
      const username = localStorage.getItem('username');
      const key1 = localStorage.getItem('key');
  
      if (!CID) {
        errorMessage = 'Company ID not found in local storage.';
        return;
      }
  
      if (newPassword !== confirmPassword) {
        errorMessage = 'Passwords do not match!';
        return;
      }
  
      if (newPassword.length < 8) {
        errorMessage = 'Password must be at least 8 characters long.';
        return;
      }
  
      try {
        const encryptedPassword = await encryptPassword(newPassword, key1 || '');
  
        const requestBody = {
          CID: CID,
          Password: encryptedPassword,
          UserName: username,
          CName: companyName,
          CAddress: companyAddress,
          CLogo: companyLogo,
          LastModifiedBy: 'Admin'
        };
  
        const response = await fetch(
          `https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/company/update/${CID}`,
          {
            method: 'PUT',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(requestBody)
          }
        );
  
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
  
        const data = await response.json();
        if (data.message === 'Company updated successfully') {
          errorMessage = '';
          newPassword = '';
          confirmPassword = '';
          window.location.href = '/login';
          
          // You can add a success message or redirect here
        } else {
          errorMessage = 'Password update failed!';
        }
      } catch (error) {
        errorMessage = 'An error occurred while updating the password.';
        console.error(error);
      }
    };
  
    const encryptPassword = async (password: string, key: string): Promise<string> => {
      try {
        const dataBuffer = new TextEncoder().encode(password);
        const iv = generateRandomBytes(12);
        const algorithm = { name: 'AES-GCM', iv: iv };
  
        // Ensure the key is 32 bytes (256 bits) long
        const keyBuffer = new TextEncoder().encode(key);
        const paddedKey = new Uint8Array(32);
        paddedKey.set(keyBuffer.subarray(0, 32));
  
        const importedKey = await window.crypto.subtle.importKey('raw', paddedKey, algorithm, false, ['encrypt']);
  
        const encryptedData = await window.crypto.subtle.encrypt(algorithm, importedKey, dataBuffer);
        const encryptedDataWithIV = new Uint8Array(iv.byteLength + encryptedData.byteLength);
        encryptedDataWithIV.set(iv);
        encryptedDataWithIV.set(new Uint8Array(encryptedData), iv.byteLength);
        return btoa(String.fromCharCode(...new Uint8Array(encryptedDataWithIV)));
      } catch (error) {
        console.error('Encryption error:', error);
        throw error;
      }
    };
  
    const generateRandomBytes = (length: number): Uint8Array => {
      const randomValues = new Uint8Array(length);
      window.crypto.getRandomValues(randomValues);
      return randomValues;
    };
  
  </script>
  
  <div class="min-h-screen flex flex-col">
  
    <!-- Main Content -->
    <div class="flex flex-col md:flex-row flex-1">
      <!-- Left Column -->
      <div
        class="w-full md:w-1/2 bg-[#D9E9FB] flex flex-col justify-center items-center py-1 px-4"
      >
        <img
          src="/loginImage.png" 
          alt="Login"
          class="xl:w-113 md:w-42 w-60 md:pt-48 xl:mb-26 mb-24 xl:pt-42 pt-50"
        />
        <h3 class="text-center xl:text-3xl md:text-2xl text-[22px] md:pt-0 xl:pt-0 font-semibold text-gray-800 pb-2">
          Optimize Your Workforce Efficiency
        </h3>
        <p class="text-center text-lg md:text-base xl:text-base text-[15px] pb-6 text-gray-900">
          Streamlined time tracking and management for a productive workplace.
        </p>
      </div>
  
      <!-- Right Column -->
      <div class="md:w-1/2 flex justify-center items-center p-12 md:p-14 xl:pt-30 md:pt-30">
        <div
          class="w-full max-w-lg px-14 py-14 rounded-lg  shadow-[0_0_20px_rgba(0,0,0,0.1)] bg-white text-center"
        >
          <h2 class="text-2xl xl:text-3xl md:text-2xl text-gray-800 font-semibold mb-12">Password</h2>
          <form on:submit|preventDefault={updatePassword}>
            <!-- New Password Field -->
            <div class="relative mb-3 xs:mb-4 sm:mb-6">
                <input
                  id="newPassword"
                  type="password"
                  bind:value={newPassword}
                  class="w-full p-2 xs:p-3 sm:p-4 text-sm xs:text-base sm:text-lg border border-[#02066F] rounded-lg focus:outline-none focus:ring-1 focus:ring-[#02066F] peer"
                  placeholder=" "
                  required
                />
                <label
                  for="newPassword"
                  class="absolute left-2 xs:left-3 top-2 xs:top-3 sm:top-4 px-1 bg-white text-[#02066F] transition-all duration-200
                         text-xs xs:text-sm sm:text-base
                         peer-placeholder-shown:text-[#02066F] 
                         peer-placeholder-shown:top-2 xs:peer-placeholder-shown:top-3 sm:peer-placeholder-shown:top-4
                         peer-placeholder-shown:text-xs xs:peer-placeholder-shown:text-sm sm:peer-placeholder-shown:text-base
                         peer-focus:-top-2 xs:peer-focus:-top-3 sm:peer-focus:-top-3
                         peer-focus:text-2xs xs:peer-focus:text-xs sm:peer-focus:text-sm
                         peer-focus:text-[#02066F]
                         peer-not-placeholder-shown:-top-2 xs:peer-not-placeholder-shown:-top-3 sm:peer-not-placeholder-shown:-top-3
                         peer-not-placeholder-shown:text-2xs xs:peer-not-placeholder-shown:text-xs sm:peer-not-placeholder-shown:text-sm
                         pointer-events-none"
                >
                  New Password
                </label>
              </div>
              
  
            <!-- Confirm Password Field -->
            <div class="relative mb-6">
              <input
                id="confirmPassword"
                type="password"
                bind:value={confirmPassword}
                class="w-full p-3 border border-[#02066F] rounded-lg focus:outline-none focus:ring-1 focus:ring-[#02066F] peer"
                placeholder=" "
                required
              />
              <label
                for="confirmPassword"
                class="absolute left-2 xs:left-3 top-2 xs:top-3 sm:top-4 px-1 bg-white text-[#02066F] transition-all duration-200
                         text-xs xs:text-sm sm:text-base
                         peer-placeholder-shown:text-[#02066F] 
                         peer-placeholder-shown:top-2 xs:peer-placeholder-shown:top-3 sm:peer-placeholder-shown:top-4
                         peer-placeholder-shown:text-xs xs:peer-placeholder-shown:text-sm sm:peer-placeholder-shown:text-base
                         peer-focus:-top-2 xs:peer-focus:-top-3 sm:peer-focus:-top-3
                         peer-focus:text-2xs xs:peer-focus:text-xs sm:peer-focus:text-sm
                         peer-focus:text-[#02066F]
                         peer-not-placeholder-shown:-top-2 xs:peer-not-placeholder-shown:-top-3 sm:peer-not-placeholder-shown:-top-3
                         peer-not-placeholder-shown:text-2xs xs:peer-not-placeholder-shown:text-xs sm:peer-not-placeholder-shown:text-sm
                         pointer-events-none"
              >
                Confirm Password
              </label>
            </div>
  
            <!-- Error Message -->
            {#if errorMessage}
              <div class="text-red-500 mb-4">{errorMessage}</div>
            {/if}
  
            <!-- Submit Button -->
            <button
              type="button"
              on:click={updatePassword}
              class="w-full bg-[#02066F] text-white py-3 px-4 rounded-lg text-lg cursor-pointer focus:outline-none transition-colors"
            >
              Submit
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>