<script>

  
  let showOverlay = false;

  // @ts-ignore
  const validateForm = (event) => {
    const form = event.target;

    if (form.checkValidity()) {
      // Valid form — show overlay
      showOverlay = true;

      setTimeout(() => {
        showOverlay = false;
        alert("Form submitted!");
        form.reset();
        // You can manually submit the form here if needed: form.submit();
      }, 2000);
    } else {
      // Invalid form — show default browser error messages
      form.reportValidity();
    }
  };


  // Phone number formatting
  let phone = "";

// @ts-ignore
const formatPhoneNumber = (event) => {
  let value = event.target.value.replace(/\D/g, '').slice(0, 10);

  if (value.length > 6) {
    phone = `${value.slice(0, 3)}-${value.slice(3, 6)}-${value.slice(6)}`;
  } else if (value.length > 3) {
    phone = `${value.slice(0, 3)}-${value.slice(3)}`;
  } else {
    phone = value;
  }
};

  const features = [
    {
      icon: 'Mask group.png',
      title: 'Facial Recognition',
      desc: 'Snap photo to log hours instantly.',
    },
    {
      icon: 'Mask group-1.png',
      title: 'Clock In/Out',
      desc: 'Seamless one-tap login and logout solution with employee identifications.',
    },
    {
      icon: 'Mask group-2.png',
      title: 'Timesheet Reports',
      desc: 'Provides employee time reports at your preferred frequency.',
    },
    {
      icon: 'Mask group-3.png',
      title: 'Admin Dashboard',
      desc: 'Employee onboarding system for Admins.',
    },
    {
      icon: 'Mask group-4.png',
      title: 'Export Options',
      desc: 'Delivers time reports in multiple formats like CSV and PDF.',
    },
    {
      icon: 'Mask group-5.png',
      title: 'Validation',
      desc: 'Admin features to update time entries.',
    }
  ];

  let zip = '';
  let isValid = true;

  function validateZip() {
    const regex = /^\d{5}(-\d{4})?$/;
    isValid = regex.test(zip);
  }
</script>

<!-- Loading Overlay -->
{#if showOverlay}
<div class="fixed inset-0 bg-black bg-opacity-50 z-50 flex items-center justify-center p-10">
  <div class="w-12 h-12 border-4 border-white border-t-blue-600 rounded-full animate-spin"></div>
</div>
{/if}

<!-- Header -->
<section class="max-w-7xl mx-auto px-4 pt-24 text-center">
<h2 class="text-3xl font-bold text-[#02066F] mb-3">Employee Time Tracking</h2>
<p class="text-[16px]">
  One tap solution for simplifying and streamlining employee time logging and reporting.
</p>
</section>

<!-- Features Section -->
<section id="whatWeProvide" class="flex flex-col lg:flex-row p-4 pt-10 gap-12 items-center">
<img src="/main image.jpeg" alt="Main Feature" class="w-full lg:w-1/2 shadow-md" />

<div class="space-y-8 w-full lg:w-1/2">
  {#each features as { icon, title, desc }}
    <div class="flex items-start gap-6 sm:gap-8">
      <img src={`/${icon}`} alt={title} class="w-14 h-14 sm:w-18 sm:h-18 object-contain" />
      <div class="flex flex-col gap-2">
        <h4 class="text-xl sm:text-2xl font-semibold">{title}</h4>
        <p class="text-gray-800 text-base sm:text-lg">{desc}</p>
      </div>
    </div>
  {/each}
</div>
</section>

<!-- Contact Section -->
<section id="contact" class="pt-18">
<div class="max-w-2xl mx-auto px-4">
  <h2 class="text-[32px] sm:text-[40px] font-bold text-gray-900 text-center mb-2">Contact</h2>

  <form
    on:submit|preventDefault={validateForm}
    class="bg-white rounded-lg shadow-[0_0_20px_rgba(0,0,0,0.2)] p-6 sm:p-10 space-y-6"
  >
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <input
        type="text"
        placeholder="First Name"
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="text"
        placeholder="Last Name"
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="email"
        placeholder="Email"
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="text"
        bind:value={phone}
        placeholder="Phone"
        on:input={formatPhoneNumber}
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="text"
        placeholder="Street"
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="text"
        placeholder="City"
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <!-- <input
        type="text"
        placeholder="Zip"
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      /> -->

      <input
      type="text"
      bind:value={zip}
      on:input={validateZip}
      placeholder="Zip"
      class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
      required
    />

      <input
        type="text"
        placeholder="State"
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
    </div>

    <input
      type="text"
      placeholder="Subject"
      class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
      required
    />

    <textarea
      placeholder="Message..."
      rows="4"
      class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold resize-none text-center focus:outline-none"
      required
    ></textarea>

    <button
      type="submit"
      class="w-full bg-[#02066F] text-lg cursor-pointer text-white font-semibold py-4 rounded-[10px] transition-colors"
    >
      Submit
    </button>
  </form>
  <!-- {#if showOverlay}
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center text-white text-xl">
    Processing...
  </div>
{/if} -->
</div>
</section>
