<script lang="ts">
    import { goto } from '$app/navigation';

    let activePage = 'Home';
    let mobileMenuOpen = false;

    const pages = ['Home', 'Login', 'Register', 'Contact Us', 'PrivacyPolicy'];

    const getPath = (page: string) => {
        if(page === 'Contact Us')
            return page='/#contact';

        return page === 'Home' ? '/' : '/' + page.toLowerCase().replace(/\s+/g, '-');
    };

    function navigate(page: string) {
        activePage = page;
        goto(getPath(page));
    }
</script>

<!-- Header -->
<header class="fixed top-0 left-0 right-0 z-50 bg-white">
    <div class="flex justify-between items-center px-2 py-0">
        <!-- Logo -->
        <div class="flex items-center p-2 pl-1">
            <img class="w-19" src="/app-logo.png" alt="app-logo" />
        </div>

        <!-- Mobile Menu Button -->
        <button
            class="md:hidden p-2 rounded-md hover:text-[#02066F] focus:outline-none"
            on:click={() => mobileMenuOpen = true}
            aria-label="Open menu"
        >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
            </svg>
        </button>

        <!-- Desktop Navigation -->
        <nav class="hidden md:block">
            <ul class="flex space-x-12 pr-8 text-lg">
                {#each pages as page}
                    <li>
                        <button
                            class="px-2 py-1 transition-all cursor-pointer duration-200 hover:text-[#02066F] hover:underline hover:decoration-2 hover:underline-offset-12 focus:text-[#02066F] focus:underline focus:decoration-2 focus:underline-offset-12
                                {activePage === page ? 'text-[#02066F] underline decoration-2 underline-offset-12' : 'text-gray-500'}"
                            on:click={() => navigate(page)}
                        >
                            {page}
                        </button>
                    </li>
                {/each}
            </ul>
        </nav>
    </div>
    <hr class="mt-2" />
</header>

<!-- Mobile Sidebar -->
<div class="md:hidden">
    <!-- Overlay -->
    <div
        class="fixed inset-0 bg-opacity-50 z-40 transition-opacity duration-300 {mobileMenuOpen ? 'opacity-100' : 'opacity-0 pointer-events-none'}"
        on:click={() => mobileMenuOpen = false}
    ></div>

    <!-- Sidebar -->
    <aside
        class="fixed top-0 left-0 h-full w-64 bg-[#02066F] text-white shadow-lg z-50 transform transition-transform duration-300 ease-in-out {mobileMenuOpen ? 'translate-x-0' : '-translate-x-full'}"
    >
        <div class="flex justify-between items-center p-4">
            <img class="w-20 pt-2" src="/logo.png" alt="logo" />
            <button
                class="p-1 rounded-full hover:bg-gray-100 focus:outline-none"
                on:click={() => mobileMenuOpen = false}
            >
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
            </button>
        </div>

        <nav class="p-4">
            <ul class="space-y-3">
                {#each pages as page}
                    <li>
                        <button
                            class="w-full block text-left px-4 py-2 rounded transition-all duration-200 hover:decoration-2 focus:decoration-2
                                {activePage === page ? 'text-white decoration-2 font-semibold' : 'text-white'}"
                            on:click={() => {
                                navigate(page);
                                mobileMenuOpen = false;
                            }}
                        >
                            {page}
                        </button>
                    </li>
                {/each}
            </ul>
        </nav>
    </aside>
</div>

<!-- Optional spacer -->
<!-- <div class="h-16"></div> -->
