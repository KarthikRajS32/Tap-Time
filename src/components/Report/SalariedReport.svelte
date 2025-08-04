<script lang="ts">
    import { onMount } from "svelte";
    import jsPDF from "jspdf";
    // Import autoTable like this:
    import autoTable from "jspdf-autotable";

    const apiUrlBase =
        "https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/report/dateRangeReportGet";
    const deviceApiUrl =
        "https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/device";

    // State
    let startDateHeader = "";
    let endDateHeader = "";
    let reportName = "";
    export let reportTypeHeading = "";
    // @ts-ignore
    let employees = [];
    let isLoading = true;
    let searchTerm = "";
    let sortConfig = { key: "name", direction: "asc" };
    let currentPage = 1;
    let itemsPerPage = 10;
    let isShowYearMonth = true;

    // Device dropdown state
    let devices: any[] = [];
    let selectedDevice: any = null;

    let adminType = "";
    let deviceID = "";

    let selectedFrequency: string = ""; // Currently selected frequency
    let availableFrequencies: string[] = []; // Load immediately, not async

    // Synchronous function to load frequencies immediately
    function loadFrequenciesSync(): string[] {
        if (typeof window === 'undefined') return [];
        const savedFrequencies = localStorage.getItem("reportType");
        if (savedFrequencies) {
            return savedFrequencies.split(",").filter((f) => f.trim() !== "");
        }
        return [];
    }

    let currentReportType = "";

    // Computed properties
    // @ts-ignore
    $: filteredEmployees = employees
        .filter(
            (emp) =>
                emp.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
                emp.pin.toLowerCase().includes(searchTerm.toLowerCase()) ||
                emp.hoursWorked
                    .toLowerCase()
                    .includes(searchTerm.toLowerCase()),
        )
        .sort((a, b) => {
            if (a[sortConfig.key] < b[sortConfig.key]) {
                return sortConfig.direction === "asc" ? -1 : 1;
            }
            if (a[sortConfig.key] > b[sortConfig.key]) {
                return sortConfig.direction === "asc" ? 1 : -1;
            }
            return 0;
        });

    $: paginatedEmployees = filteredEmployees.slice(
        (currentPage - 1) * itemsPerPage,
        currentPage * itemsPerPage,
    );

    $: totalPages = Math.ceil(filteredEmployees.length / itemsPerPage);

    // Combine the two onMount into one

    // Fetch device data
    // (duplicate removed)

   onMount(() => {
  // Initialize browser-specific data
  adminType = localStorage.getItem("adminType") || "";
  deviceID = localStorage.getItem("DeviceID") || "";
  availableFrequencies = loadFrequenciesSync();
  
  const selectedValue = localStorage.getItem("reportType") ?? "";
  let exactSelectedValue = localStorage.getItem("selectedFrequency"); 
  currentReportType = selectedValue;
  reportName = `${selectedValue} Report`;
  reportTypeHeading = `${exactSelectedValue} Report`;

  // Get selected frequency
  const selectedFreq = localStorage.getItem("selectedFrequency");

  // Get available frequencies
  const savedFrequencies = localStorage.getItem("reportType");
  if (savedFrequencies) {
    availableFrequencies = savedFrequencies.split(",");
    
    // If selected frequency exists in list, set it
    if (selectedFreq && availableFrequencies.includes(selectedFreq)) {
      selectedFrequency = selectedFreq;
    } else {
      selectedFrequency = availableFrequencies[0];
    }

    // Generate and load initial table
    dateRanges = generateDateRanges();
    if (dateRanges.length > 0) {
      loadReportTable(dateRanges[0].startRange, dateRanges[0].endRange);
    }
  }

  toggleSelectors();
  viewDateRangewiseReport();
  fetchDevices();
});


    // Fetch device data
    async function fetchDevices() {
        try {
            if (typeof window === 'undefined') return;
            const cid = localStorage.getItem("companyID");
            const response = await fetch(`${deviceApiUrl}/getAll/${cid}`);
            if (!response.ok) throw new Error(`Error: ${response.status}`);

            const data = await response.json();
            // Filter out devices with "Not Registered" names
            const allDevices = Array.isArray(data) ? data : [data];
            devices = allDevices.filter(
                (device) =>
                    device.DeviceName &&
                    device.DeviceName !== "Not Registered" &&
                    device.DeviceName.trim() !== "",
            );

            // Set first valid device as default selection
            if (devices.length > 0) {
                selectedDevice = devices[0];
                console.log("Default selected device:", selectedDevice);
            }
        } catch (error) {
            console.error("Error fetching devices:", error);
        }
    }

    // Handle device selection
    const handleDeviceSelection = (device: any) => {
        selectedDevice = device;
        console.log("Selected device for salaried reports:", device);

        // Refresh report with new device filter using currently selected date range
        if (dateRanges.length > 0 && selectedRangeIndex < dateRanges.length) {
            const currentRange = dateRanges[selectedRangeIndex];
            console.log(
                `Refreshing report for device ${device.DeviceName}, range: ${currentRange.startRange} to ${currentRange.endRange}`,
            );
            loadReportTable(currentRange.startRange, currentRange.endRange);
        } else if (dateRanges.length > 0) {
            // Fallback to first range if selectedRangeIndex is invalid
            console.log(
                `Using first range for device ${device.DeviceName}, range: ${dateRanges[0].startRange} to ${dateRanges[0].endRange}`,
            );
            selectedRangeIndex = 0;
            loadReportTable(dateRanges[0].startRange, dateRanges[0].endRange);
        }
    };

    // Function to switch between frequencies
    function switchFrequency(frequency: string) {
        selectedFrequency = frequency;
        reportName = `${frequency} Report`;
        reportTypeHeading = `${frequency} Report`;
        currentReportType = frequency;

        // Reset to current year/month when switching frequencies
        year = new Date().getFullYear();
        month = new Date().getMonth() + 1;
        week = 1;
        half = "first";

        // Regenerate date ranges
        dateRanges = generateDateRanges();
        if (dateRanges.length > 0) {
            loadReportTable(dateRanges[0].startRange, dateRanges[0].endRange);
        }

        // Update UI selectors
        toggleSelectors();
        updateDates();
    }

    // @ts-ignore
    function requestSort(key) {
        let direction = "asc";
        if (sortConfig.key === key && sortConfig.direction === "asc") {
            direction = "desc";
        }
        sortConfig = { key, direction };
        currentPage = 1; // Reset to first page when sorting
    }

    async function loadReportData() {
        isLoading = true;
        if (typeof window === 'undefined') return;
        const cid = localStorage.getItem("companyID");
        const dateRange = getDateRange(selectedFrequency); // Pass selected frequency

        startDateHeader = dateRange.startRange;
        endDateHeader = dateRange.endRange;

        try {
            const response = await fetch(
                `${apiUrlBase}/${cid}/${startDateHeader}/${endDateHeader}`,
            );
            if (!response.ok) throw new Error(`HTTP error: ${response.status}`);

            const data = await response.json();
            employees = Object.entries(calculateTotalTimeWorked(data)).map(
                ([pin, empData]) => {
                    const { name, totalHoursWorked } = empData as {
                        name: string;
                        totalHoursWorked: string;
                    };
                    return {
                        pin,
                        name,
                        hoursWorked: totalHoursWorked,
                    };
                },
            );
        } catch (error) {
            console.error("Failed to load report:", error);
            employees = [];
        } finally {
            isLoading = false;
        }
    }

    // function getDateRange(frequency: string) {
    //     switch (frequency) {
    //         case "Weekly": return getLastWeekDateRange();
    //         case "Monthly": return getLastMonthStartAndEndDates();
    //         case "Bimonthly": return getLastMonthStartAndEndDates();
    //         case "Biweekly": return getLastTwoWeeksDateRange();
    //         default: return { startRange: '', endRange: '' };
    //     }
    // }

    function getDateRange(frequency: string) {
        // Get first and last day of selected month
        const firstDay = new Date(year, month - 1, 1);
        const lastDay = new Date(year, month, 0);

        switch (frequency) {
            case "Weekly":
                // Calculate weeks for selected month
                const weeksInMonth = Math.ceil(
                    (lastDay.getDate() - firstDay.getDay() + 1) / 7,
                );
                const selectedWeek = Math.min(week, weeksInMonth);

                const weekStart = new Date(
                    year,
                    month - 1,
                    1 + (selectedWeek - 1) * 7 - firstDay.getDay(),
                );
                const weekEnd = new Date(weekStart);
                weekEnd.setDate(weekStart.getDate() + 6);

                // Ensure dates stay within month
                if (weekStart < firstDay) weekStart.setDate(1);
                if (weekEnd > lastDay) weekEnd.setDate(lastDay.getDate());

                return {
                    startRange: formatDate(weekStart),
                    endRange: formatDate(weekEnd),
                };

            case "Biweekly":
                // Find first Sunday of month
                let firstSunday = new Date(firstDay);
                firstSunday.setDate(
                    firstDay.getDate() + ((7 - firstDay.getDay()) % 7),
                );

                // Calculate selected period
                let periodStart = new Date(firstSunday);
                if (week === 2) {
                    periodStart.setDate(firstSunday.getDate() + 14);
                }

                const periodEnd = new Date(periodStart);
                periodEnd.setDate(periodStart.getDate() + 13);

                // Ensure dates stay within month
                if (periodStart < firstDay) periodStart = new Date(firstDay);
                if (periodEnd > lastDay) periodEnd.setDate(lastDay.getDate());

                return {
                    startRange: formatDate(periodStart),
                    endRange: formatDate(periodEnd),
                };

            case "Bimonthly":
                const daysInMonth = lastDay.getDate();
                const mid = Math.ceil(daysInMonth / 2);

                if (half === "first") {
                    return {
                        startRange: `${year}-${pad(month)}-01`,
                        endRange: `${year}-${pad(month)}-${pad(mid)}`,
                    };
                } else {
                    return {
                        startRange: `${year}-${pad(month)}-${pad(mid + 1)}`,
                        endRange: `${year}-${pad(month)}-${pad(daysInMonth)}`,
                    };
                }

            case "Monthly":
                return {
                    startRange: `${year}-${pad(month)}-01`,
                    endRange: `${year}-${pad(month)}-${pad(lastDay.getDate())}`,
                };

            default:
                return { startRange: "", endRange: "" };
        }
    }

    function getLastWeekDateRange() {
        const now = new Date();
        const dayOfWeek = now.getDay(); // 0 (Sun) - 6 (Sat)
        const lastSunday = new Date(now);
        lastSunday.setDate(now.getDate() - dayOfWeek - 7);
        const lastSaturday = new Date(now);
        lastSaturday.setDate(now.getDate() - dayOfWeek - 1);
        return {
            startRange: formatDate(lastSunday),
            endRange: formatDate(lastSaturday),
        };
    }

    function getLastMonthStartAndEndDates() {
        const now = new Date();
        const start = new Date(now.getFullYear(), now.getMonth() - 1, 1);
        const end = new Date(now.getFullYear(), now.getMonth(), 0);
        return {
            startRange: formatDate(start),
            endRange: formatDate(end),
        };
    }

    function getLastTwoWeeksDateRange() {
        const now = new Date();
        const dayOfWeek = now.getDay(); // 0 (Sun) - 6 (Sat)
        // End date is last Saturday
        const endDate = new Date(now);
        endDate.setDate(now.getDate() - dayOfWeek - 1);
        // Start date is two Sundays before (13 days before end date)
        const startDate = new Date(endDate);
        startDate.setDate(endDate.getDate() - 13);
        return {
            startRange: formatDate(startDate),
            endRange: formatDate(endDate),
        };
    }

    // @ts-ignore
    function formatDate(date) {
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, "0");
        const day = String(date.getDate()).padStart(2, "0");
        return `${year}-${month}-${day}`;
    }

    // @ts-ignore
    function calculateTotalTimeWorked(data) {
        const employeeTimes = {};

        // @ts-ignore
        data.forEach((entry) => {
            const { Name, Pin, CheckInTime, CheckOutTime } = entry;

            if (!Name || !Pin || !CheckInTime) return;

            const checkInDate = new Date(CheckInTime);
            const checkOutDate = CheckOutTime
                ? new Date(CheckOutTime)
                : new Date();
            // @ts-ignore
            const timeDifferenceInMinutes = Math.floor(
                (Number(checkOutDate) - Number(checkInDate)) / 1000 / 60,
            );

            // @ts-ignore
            if (!employeeTimes[Pin]) {
                // @ts-ignore
                employeeTimes[Pin] = { name: Name, totalMinutes: 0 };
            }

            // @ts-ignore
            employeeTimes[Pin].totalMinutes += timeDifferenceInMinutes;
        });

        for (const [pin, details] of Object.entries(employeeTimes) as [
            string,
            { name: string; totalMinutes: number; totalHoursWorked?: string },
        ][]) {
            details.totalHoursWorked = minutesToTime(details.totalMinutes);
        }

        return employeeTimes;
    }

    // @ts-ignore
    function minutesToTime(minutes) {
        const hours = Math.floor(minutes / 60);
        const mins = minutes % 60;
        return `${hours}:${mins.toString().padStart(2, "0")}`;
    }

    function downloadPDF() {
        if (filteredEmployees.length === 0) {
            alert("No data to download");
            return;
        }

        const doc = new jsPDF();
        const formattedDateRange = `${startDateHeader} to ${endDateHeader}`;

        doc.setFontSize(14);
        doc.text(`${reportTypeHeading} (${formattedDateRange})`, 14, 10);

        const tableColumn = ["Name", "Pin", "Time Worked Hours (HH:MM)"];
        const tableRows = filteredEmployees.map((emp) => [
            emp.name || "--",
            emp.pin || "--",
            emp.hoursWorked || "--",
        ]);

        autoTable(doc, {
            head: [tableColumn],
            body: tableRows,
            startY: 20,
            headStyles: {
                fillColor: [2, 6, 111],
                textColor: 255,
                fontSize: 10,
                fontStyle: "bold",
            },
            styles: { fontSize: 10 },
            theme: "grid",
        });

        const fileName = `${reportTypeHeading.toLowerCase().replace(/\s+/g, "_")}.pdf`;
        doc.save(fileName);
    }

    function downloadCSV() {
        if (filteredEmployees.length === 0) {
            alert("No data to download");
            return;
        }

        const headers = [
            "Employee Name",
            "Employee ID",
            "Total Worked Hours (HH:MM)",
        ];
        const csvData = filteredEmployees.map((employee) => [
            employee.name,
            employee.pin,
            employee.hoursWorked,
        ]);

        let csvContent = headers.join(",") + "\n";
        csvData.forEach((row) => {
            csvContent += row.join(",") + "\n";
        });

        const blob = new Blob([csvContent], { type: "text/csv" });
        const url = window.URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.download = `${reportName.toLowerCase().replace(" ", "_")}_${startDateHeader}_to_${endDateHeader}.csv`;
        a.click();
        window.URL.revokeObjectURL(url);
    }

    // Types
    type Employee = {
        name: string;
        pin: string;
        hoursWorked: string;
    };

    type DateRange = {
        startRange: string;
        endRange: string;
        label?: string;
    };

    type EmployeeTimeData = {
        name: string;
        totalMinutes: number;
        totalHoursWorked?: string;
    };

    const months = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
    ];

    // State
    let reportData: any[] = [];
    let dateRanges: DateRange[] = [];
    let showDownloadButtons = false;
    let selectedRangeIndex = 0;
    let showWeekSelector = false;
    let showHalfSelector = false;
    let year = new Date().getFullYear();
    let month = new Date().getMonth() + 1;
    let week = 1;
    let half = "first";

    // This is now handled in the main onMount above
    // onMount(() => {
    //     const selectedValue = localStorage.getItem("reportType");
    //     reportName = `${selectedValue} Report`;
    //     reportTypeHeading = `${selectedValue} Report`;
    //     toggleSelectors();
    //     viewDateRangewiseReport();
    //     fetchDevices();
    // });

    function toggleSelectors() {
        if (typeof window === 'undefined') return;
        const reportType = localStorage.getItem("reportType");
        showWeekSelector = reportType === "Weekly";
        showHalfSelector = reportType === "Bimonthly";
    }

    function pad(n: number): string {
        return n.toString().padStart(2, "0");
    }

    // Helper for short date display (06 Jan)
    function formatShortDate(date: Date): string {
        const day = date.getDate().toString().padStart(2, "0");
        const monthNames = [
            "Jan",
            "Feb",
            "Mar",
            "Apr",
            "May",
            "Jun",
            "Jul",
            "Aug",
            "Sep",
            "Oct",
            "Nov",
            "Dec",
        ];
        return `${day} ${monthNames[date.getMonth()]}`;
    }

    function generateDateRanges(): DateRange[] {
        const ranges: DateRange[] = [];
        const reportType =
            currentReportType || (typeof window !== 'undefined' ? localStorage.getItem("reportType") : '');

        // if (reportType === "Weekly") {
        //     const daysInMonth = new Date(year, month, 0).getDate(); // Jan = 1
        //     const totalWeeks = Math.ceil(daysInMonth / 7);

        //     for (let week = 0; week < totalWeeks; week++) {
        //         const startDay = week * 7 + 1;
        //         let endDay = startDay + 6;

        //         // Don't go past month end
        //         if (endDay > daysInMonth) {
        //             endDay = daysInMonth;
        //         }

        //         ranges.push({
        //             startRange: `${year}-${pad(month)}-${pad(startDay)}`,
        //             endRange: `${year}-${pad(month)}-${pad(endDay)}`,
        //             label: `Report ${week + 1}: ${year}-${pad(month)}-${pad(startDay)} - ${year}-${pad(month)}-${pad(endDay)}`
        //         });
        //     }
        // }

        if (reportType === "Weekly" || selectedFrequency === "Weekly") {
            // Predefined week ranges for specific months
            const monthWeekRanges: Record<
                number,
                Array<{ start: number; end: number }>
            > = {
                1: [
                    { start: 6, end: 12 },
                    { start: 13, end: 19 },
                    { start: 20, end: 26 },
                ], // January
                2: [
                    { start: 3, end: 9 },
                    { start: 10, end: 16 },
                    { start: 17, end: 23 },
                ], // February
                3: [
                    { start: 3, end: 9 },
                    { start: 10, end: 16 },
                    { start: 17, end: 23 },
                    { start: 24, end: 30 },
                ], // March
                4: [
                    { start: 7, end: 13 },
                    { start: 14, end: 20 },
                    { start: 21, end: 27 },
                ], // April
                5: [
                    { start: 5, end: 11 },
                    { start: 12, end: 18 },
                    { start: 19, end: 25 },
                ], // May
                6: [
                    { start: 2, end: 8 },
                    { start: 9, end: 15 },
                    { start: 16, end: 22 },
                    { start: 23, end: 29 },
                ], //June
                7: [
                    { start: 7, end: 13 },
                    { start: 14, end: 20 },
                    { start: 21, end: 27 },
                ], //July
                8: [
                    { start: 4, end: 10 },
                    { start: 11, end: 17 },
                    { start: 18, end: 24 },
                    { start: 25, end: 31 },
                ], //August
                9: [
                    { start: 1, end: 7 },
                    { start: 8, end: 14 },
                    { start: 15, end: 21 },
                    { start: 22, end: 28 },
                ], //September
                10: [
                    { start: 6, end: 12 },
                    { start: 13, end: 19 },
                    { start: 20, end: 26 },
                ], //October
                11: [
                    { start: 3, end: 9 },
                    { start: 10, end: 16 },
                    { start: 17, end: 23 },
                    { start: 24, end: 30 },
                ], //November
                12: [
                    { start: 1, end: 7 },
                    { start: 8, end: 14 },
                    { start: 15, end: 21 },
                    { start: 22, end: 28 },
                ], //December
                // Add other months as needed
            };

            const daysInMonth = new Date(year, month, 0).getDate();
            const monthRanges = monthWeekRanges[month] || [];

            for (let i = 0; i < monthRanges.length; i++) {
                const { start, end } = monthRanges[i];
                const startDate = new Date(
                    year,
                    month - 1,
                    Math.min(start, daysInMonth),
                );
                const endDate = new Date(
                    year,
                    month - 1,
                    Math.min(end, daysInMonth),
                );

                ranges.push({
                    startRange: formatDate(startDate),
                    endRange: formatDate(endDate),
                    label: `${formatShortDate(startDate)} to ${formatShortDate(endDate)}`,
                });
            }
        }
        if (reportType === "Bimonthly" || selectedFrequency === "Bimonthly") {
            isShowYearMonth = true;
            const daysInMonth = new Date(year, month, 0).getDate();
            const mid = Math.ceil(daysInMonth / 2);

            // Report 1 (previously First Half)
            ranges.push({
                startRange: `${year}-${pad(month)}-01`,
                endRange: `${year}-${pad(month)}-${pad(mid)}`,
                label: `Report 1: ${year}-${pad(month)}-01 - ${year}-${pad(month)}-${pad(mid)}`,
            });

            // Report 2 (previously Second Half)
            ranges.push({
                startRange: `${year}-${pad(month)}-${pad(mid + 1)}`,
                endRange: `${year}-${pad(month)}-${pad(daysInMonth)}`,
                label: `Report 2: ${year}-${pad(month)}-${pad(mid + 1)} - ${year}-${pad(month)}-${pad(daysInMonth)}`,
            });
        } else if (
            reportType === "Monthly" ||
            selectedFrequency === "Monthly"
        ) {
            isShowYearMonth = true;
            const daysInMonth = new Date(year, month, 0).getDate();
            ranges.push({
                startRange: `${year}-${pad(month)}-01`,
                endRange: `${year}-${pad(month)}-${pad(daysInMonth)}`,
                label: "Full Month",
            });
        } else if (
            reportType === "Biweekly" ||
            selectedFrequency === "Biweekly"
        ) {
            isShowYearMonth = false;
            const today = new Date();
            const dayOfWeek = today.getDay(); // Sunday = 0

            // Get last week's Sunday
            const lastSunday = new Date(today);
            lastSunday.setDate(today.getDate() - dayOfWeek - 7);

            // Today is the end date
            const currentDate = new Date();

            // Push range
            ranges.push({
                startRange: formatDate(lastSunday),
                endRange: formatDate(currentDate),
                label: `Last Sunday to Today: ${formatDate(lastSunday)} - ${formatDate(currentDate)}`,
            });
        }

        return ranges;
    }

    async function loadReportTable(startVal: string, endVal: string) {
        isLoading = true;
        startDateHeader = startVal;
        endDateHeader = endVal;

        if (typeof window === 'undefined') return;
        const cid = localStorage.getItem("companyID");
        const deviceId = selectedDevice ? selectedDevice.DeviceID : "all";

        console.log(
            `Making fresh API call for ${startVal} to ${endVal}, device: ${deviceId}`,
        );

        try {
            const response = await fetch(
                `${apiUrlBase}/${cid}/${startVal}/${endVal}`,
            );
            const data = await response.json();

            if (Array.isArray(data)) {
                let filteredData = data;

                if (adminType != "Owner") {
                    filteredData = data.filter((item) => {
                        console.log(
                            `Record DeviceID: ${item.DeviceID}, matches: ${item.DeviceID === deviceID}`,
                        );
                        return item.DeviceID === deviceID;
                    });
                }

                // Filter by selected device if available
                else if (selectedDevice && selectedDevice.DeviceID) {
                    filteredData = data.filter((item) => {
                        console.log(
                            `Record DeviceID: ${item.DeviceID}, matches: ${item.DeviceID === selectedDevice.DeviceID}`,
                        );
                        return item.DeviceID === selectedDevice.DeviceID;
                    });
                } else {
                    console.log(
                        `No device selected, showing all records:`,
                        data.length,
                    );
                }

                reportData = filteredData;
                console.log(
                    `API call completed for ${startVal} to ${endVal}, device: ${deviceId}:`,
                    filteredData.length,
                    "records",
                );
                employees = Object.entries(
                    calculateTotalTimeWorked(filteredData),
                ).map(([pin, empData]) => {
                    const { name, totalHoursWorked } =
                        empData as EmployeeTimeData;
                    return {
                        pin,
                        name,
                        hoursWorked: totalHoursWorked || "0:00",
                    };
                });
                showDownloadButtons = filteredData.length > 0;
            } else {
                reportData = [];
                employees = [];
                showDownloadButtons = false;
            }
        } catch (error) {
            console.error("Error fetching report data", error);
            reportData = [];
            employees = [];
            showDownloadButtons = false;
        } finally {
            isLoading = false;
        }
    }

    function updateDates() {
        if (typeof window === 'undefined') return;
        const reportType = localStorage.getItem("reportType");
        const selectedRange = dateRanges[selectedRangeIndex];

        if (selectedRange) {
            startDateHeader = selectedRange.startRange;
            endDateHeader = selectedRange.endRange;
        } else if (reportType === "Monthly") {
            const daysInMonth = new Date(year, month, 0).getDate();
            startDateHeader = `${year}-${pad(month)}-01`;
            endDateHeader = `${year}-${pad(month)}-${pad(daysInMonth)}`;
        } else if (reportType === "Biweekly") {
            const today = new Date();
            const end = new Date(today);
            const start = new Date(today);
            start.setDate(end.getDate() - 13);
            startDateHeader = formatDate(start);
            endDateHeader = formatDate(end);
        }
    }

    // Modify viewDateRangewiseReport to call updateDates
    function viewDateRangewiseReport() {
        dateRanges = generateDateRanges();

        if (dateRanges.length > 0) {
            loadReportTable(dateRanges[0].startRange, dateRanges[0].endRange);
        }
        updateDates();
    }

    // Modify selectDateRange to call updateDates
    function selectDateRange(index: number) {
        selectedRangeIndex = index;
        loadReportTable(
            dateRanges[index].startRange,
            dateRanges[index].endRange,
        );
        updateDates(); // Add this line
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

<div class="bg-gray-100">
    <div class="pt-16 md:pt-18 sm:pt-2">
        {#if isLoading}
            <div
                class="fixed inset-0 flex items-center justify-center z-50"
                style="background: rgba(0, 0, 0, 0.5)"
            >
                <div
                    class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"
                ></div>
            </div>
        {/if}

        <!-- Navigation -->
        <nav class="bg-white shadow">
            <div class="max-w-8xl mx-auto px-4 sm:px-6 lg:px-8">
                <div
                    class="flex flex-col md:flex-row justify-between items-center h-auto md:h-16 py-4 md:py-0 justify-end"
                >
                    <!-- Menu Links - visible on all devices -->
                    <div
                        class="flex flex-col md:flex-row space-y-2 md:space-y-0 md:space-x-4 text-center w-auto md:w-auto"
                    >
                        <a
                            href="/reportsummary"
                            class="px-4 py-2 text-[#02066F] font-semibold rounded-full"
                            >Today's Report</a
                        >
                        <a
                            href="/daywisereport"
                            class="px-4 py-2 text-[#02066F] font-semibold rounded-full"
                            >Day Wise Report</a
                        >

                        {#if availableFrequencies.length > 1}
                            <!-- Show separate frequency buttons in nav when multiple frequencies are selected -->
                            <div class="flex flex-wrap gap-2">
                                {#each availableFrequencies as frequency}
                                    <a
                                        href="/salariedreport"
                                        on:click|preventDefault={() =>
                                            switchFrequency(frequency)}
                                        class={`px-4 py-2 rounded-full font-semibold transition-colors
                                    ${
                                        selectedFrequency === frequency
                                            ? "bg-[#02066F] text-white"
                                            : "bg-white text-[#02066F] "
                                    }
                                    text-sm sm:text-base`}
                                    >
                                        {frequency} Report
                                    </a>
                                {/each}
                            </div>
                        {:else}
                            <!-- Show single salaried report link when only one frequency is selected -->
                            <a
                                href="/salariedreport"
                                class="px-4 py-2 bg-[#02066F] text-white font-semibold rounded-full"
                            >
                                <!-- Salaried Report -->
                                {reportTypeHeading}
                            </a>
                        {/if}
                    </div>
                </div>
            </div>
        </nav>

        <!-- Device Dropdown Section -->
        <div class="max-w-5xl mx-auto mt-4 px-4">
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
                                    <div
                                        class="text-gray-500 block px-4 py-2 text-sm"
                                    >
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

        <!-- Main Content -->
        <main class="container mx-auto px-4 py-8">
            <!-- Frequency Selector Tabs -->
            <!-- {#if availableFrequencies.length > 1}
            <div class="flex flex-wrap justify-center gap-2 mb-6">
                {#each availableFrequencies as frequency}
                    <button
                        on:click={() => switchFrequency(frequency)}
                        class={`px-4 py-2 rounded-lg font-medium transition-colors
                            ${selectedFrequency === frequency
                                ? 'bg-[#02066F] text-white'
                                : 'bg-white text-[#02066F] border border-[#02066F] hover:bg-gray-100'}
                            text-sm sm:text-base`}
                    >
                        {frequency} Report
                    </button>
                {/each}
            </div>
        {/if} -->

            <h1 class="text-2xl font-bold text-center mb-8">
                {reportTypeHeading}
            </h1>

            <!-- Date Selection Controls -->
            <div class="flex flex-wrap justify-center gap-4 mb-3">
                {#if isShowYearMonth}
                <div class="flex items-center">
                    <label
                        for="yearInput"
                        class="mr-2 text-base font-semibold text-gray-800"
                        >Year:</label
                    >
                    <select
                        id="yearInput"
                        bind:value={year}
                        class="bg-white border border-[#02066F] rounded px-3 py-1 text-[#02066F] font-medium focus:outline-none"
                        on:change={() => {
                            toggleSelectors();
                            if (showWeekSelector) week = 1;
                            viewDateRangewiseReport();
                            updateDates();
                        }}
                    >
                        {#each Array.from({ length: 1 }, (_, i) => 2025 + i) as y}
                            <option value={y}>{y}</option>
                        {/each}
                    </select>
                </div>

                <div class="flex items-center">
                    <label
                        for="monthInput"
                        class="mr-2 text-base font-semibold text-gray-800"
                        >Month:</label
                    >
                    <select
                        id="monthInput"
                        bind:value={month}
                        class="bg-white border border-[#02066F] rounded px-3 py-1 text-[#02066F] font-medium focus:outline-none"
                        on:change={() => {
                            toggleSelectors();
                            // Always reset to week 1 when month changes
                            week = 1;
                            selectedRangeIndex = 0;
                            viewDateRangewiseReport();
                            updateDates();
                        }}
                    >
                        {#each months as monthName, index}
                            <option value={index + 1}>{monthName}</option>
                        {/each}
                    </select>
                </div>
                {/if}

                <!-- {#if showWeekSelector}
                <div class="flex items-center">
                    <label for="weekInput" class="mr-2 text-base font-semibold text-gray-800">Week:</label>
                    <select 
                        id="weekInput" 
                        bind:value={week}
                        class="bg-white border border-[#02066F] rounded px-3 py-1 text-[#02066F] font-medium focus:outline-none"
                        on:change={() => {
                            // When week changes, select the corresponding report
                            selectedRangeIndex = week - 1;
                            viewDateRangewiseReport();
                            updateDates(); // Add this line
                        }}
                    >
                        {#each [1, 2, 3, 4, 5] as weekNum}
                            <option value={weekNum}>Week {weekNum}</option>
                        {/each}
                    </select>
                </div>
            {/if} -->

                {#if showWeekSelector || selectedFrequency === "Weekly"}
                    <div class="flex items-center">
                        <label
                            for="weekInput"
                            class="mr-2 text-base font-semibold text-gray-800"
                            >Week:</label
                        >
                        <select
                            id="weekInput"
                            bind:value={week}
                            class="bg-white border border-[#02066F] rounded px-3 py-1 text-[#02066F] font-medium focus:outline-none"
                            on:change={() => {
                                selectedRangeIndex = week - 1;
                                loadReportTable(
                                    dateRanges[selectedRangeIndex].startRange,
                                    dateRanges[selectedRangeIndex].endRange,
                                );
                            }}
                        >
                            {#each dateRanges as range, index}
                                <option value={index + 1}>{range.label}</option>
                            {/each}
                        </select>
                    </div>
                {/if}
                {#if showHalfSelector || selectedFrequency === "Bimonthly"}
                    <div class="flex items-center">
                        <label
                            for="halfInput"
                            class="mr-2 text-base font-semibold text-gray-800"
                            >Half:</label
                        >
                        <select
                            id="halfInput"
                            bind:value={half}
                            class="bg-white border border-[#02066F] rounded px-3 py-1 text-[#02066F] font-medium focus:outline-none"
                            on:change={() => {
                                selectedRangeIndex = half === "first" ? 0 : 1;
                                viewDateRangewiseReport();
                                updateDates();
                            }}
                        >
                            <option value="first">First Half</option>
                            <option value="second">Second Half</option>
                        </select>
                    </div>
                {/if}
            </div>

            <!-- Date Range Buttons -->
            <!-- {#if dateRanges.length > 1}
            <div class="flex flex-wrap justify-center   gap-2 mb-6">
                {#each dateRanges as range, index}
                    <button
                        on:click={() => {
                            selectedRangeIndex = index;
                            // Update week/half selection to match the selected report
                            if (showWeekSelector) week = index + 1;
                            if (showHalfSelector) half = index === 0 ? 'first' : 'second';
                            loadReportTable(range.startRange, range.endRange);
                            updateDates(); // Add this line
                        }}
                        class={`px-4 py-3 text-sm md:text-md rounded border-2 border-[#02066F] font-medium transition-colors
                            ${selectedRangeIndex === index 
                                ? 'bg-[#02066F] text-white cursor-pointer' 
                                : 'bg-white text-[#02066F] cursor-pointer'}`}
                    >
                          {range.label}
                    </button>
                {/each}
            </div>
        {/if} -->

            <!-- Date Range Display -->
            <div
                class="flex flex-col max-w-5xl mx-auto md:flex-row justify-between mb-6 p-4 rounded-lg"
            >
                <div class="mb-2 md:mb-0">
                    <span class="text-md md:text-lg font-semibold text-gray-800"
                        >Start Date:
                    </span>
                    <span class="text-md md:text-lg font-semibold text-gray-800"
                        >{startDateHeader}</span
                    >
                </div>
                <div>
                    <span class="text-md md:text-lg font-semibold text-gray-800"
                        >End Date:
                    </span>
                    <span class="text-md md:text-lg font-semibold text-gray-800"
                        >{endDateHeader}</span
                    >
                </div>
            </div>

            <!-- Search and Controls -->
            <div
                class="max-w-5xl mx-auto bg-white rounded-xl shadow-sm overflow-hidden mb-8 border-1 border-gray-300"
            >
                <div class="p-4 sm:p-6 overflow-x-auto">
                    <!-- Download Buttons -->
                    <!-- {#if filteredEmployees.length > 0} -->
                    <div
                        class="flex flex-col gap-2 sm:flex-row justify-evenly items-center mb-6"
                    >
                        <button
                            on:click={downloadPDF}
                            class="text-[#02066F] hover:text-black px-4 py-2 rounded-lg transition-colors cursor-pointer border-1 border-[#02066F]"
                        >
                            Download PDF
                        </button>
                        <button
                            on:click={downloadCSV}
                            class="text-[#02066F] hover:text-black px-4 py-2 rounded-lg transition-colors cursor-pointer border-1 border-[#02066F]"
                        >
                            Download CSV
                        </button>
                    </div>
                    <!-- {/if} -->

                    <!-- Search and Entries Selector -->
                    <div
                        class="flex flex-col sm:flex-row justify-between items-center gap-4 mb-4"
                    >
                        <div class="flex items-center gap-2">
                            <label
                                for="entries"
                                class="text-base font-semibold text-gray-700"
                                >Show:</label
                            >
                            <select
                                id="entries"
                                bind:value={itemsPerPage}
                                class="border border-gray-400 rounded-md px-2 py-1 text-sm focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                            >
                                <option value="10">10</option>
                                <option value="25">25</option>
                                <option value="50">50</option>
                                <option value="100">100</option>
                            </select>
                            <span class="text-base font-semibold text-gray-700"
                                >entries</span
                            >
                        </div>

                        <div
                            class="mb-4 flex flex-col sm:flex-row sm:items-center sm:justify-end gap-2"
                        >
                            <label
                                for="search"
                                class="text-base font-semibold text-gray-800"
                                >Search:</label
                            >
                            <input
                                id="search"
                                type="text"
                                bind:value={searchTerm}
                                placeholder=""
                                class="w-full sm:w-64 px-2 py-1 border border-gray-500 rounded-md focus:outline-none focus:ring-1 focus:ring-[#02066F]"
                            />
                        </div>
                    </div>

                    <!-- Employee Table -->
                    <div class="overflow-hidden">
                        <!-- {#if filteredEmployees.length <= 0}
                        <div class="text-center p-8 text-gray-700">
                            {searchTerm ? 'No matching employees found' : 'No records available'}
                        </div>
                    {/if} -->
                        <!-- {:else} -->
                        <div
                            class="overflow-x-auto w-full max-w-full sm:rounded-lg"
                        >
                            <table
                                class="min-w-[600px] sm:min-w-full border border-gray-300 text-sm"
                            >
                                <thead class="bg-[#02066F] text-white">
                                    <tr>
                                        <th
                                            class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer border-r"
                                            on:click={() => requestSort("name")}
                                        >
                                            <div
                                                class="flex items-center justify-center"
                                            >
                                                Name
                                                {#if sortConfig.key === "name"}
                                                    <span class="ml-6 text-lg"
                                                        >{sortConfig.direction ===
                                                        "asc"
                                                            ? "↑"
                                                            : "↓"}</span
                                                    >
                                                {:else}
                                                    <span class="ml-6 text-lg"
                                                        >↑↓</span
                                                    >
                                                {/if}
                                            </div>
                                        </th>
                                        <th
                                            class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer border-r"
                                            on:click={() => requestSort("pin")}
                                        >
                                            <div
                                                class="flex items-center justify-center"
                                            >
                                                Pin
                                                {#if sortConfig.key === "pin"}
                                                    <span class="ml-1 text-lg"
                                                        >{sortConfig.direction ===
                                                        "asc"
                                                            ? "↑"
                                                            : "↓"}</span
                                                    >
                                                {:else}
                                                    <span class="ml-6 text-lg"
                                                        >↑↓</span
                                                    >
                                                {/if}
                                            </div>
                                        </th>
                                        <th
                                            class="px-6 py-3 text-center text-base font-bold tracking-wider cursor-pointer border-r"
                                            on:click={() =>
                                                requestSort("hoursWorked")}
                                        >
                                            <div
                                                class="flex items-center justify-center"
                                            >
                                                Total Worked Hours (HH:MM)
                                                {#if sortConfig.key === "hoursWorked"}
                                                    <span class="ml-1 text-lg"
                                                        >{sortConfig.direction ===
                                                        "asc"
                                                            ? "↑"
                                                            : "↓"}</span
                                                    >
                                                {:else}
                                                    <span class="ml-6 text-lg"
                                                        >↑↓</span
                                                    >
                                                {/if}
                                            </div>
                                        </th>
                                    </tr>
                                </thead>
                                <tbody
                                    class="bg-white divide-y divide-gray-200"
                                >
                                    {#if filteredEmployees.length === 0}
                                        <!-- <div class="px-4 py-4 text-center items-center justify-center text-gray-500"> -->
                                        <tr>
                                            <td
                                                colspan="5"
                                                class="px-4 py-4 text-center text-gray-500"
                                            >
                                                No matching records found
                                            </td>
                                        </tr>
                                        <!-- </div> -->
                                    {:else}
                                        {#each paginatedEmployees as employee}
                                            <tr
                                                class="hover:bg-gray-50 text-center"
                                            >
                                                <td
                                                    class="px-6 py-3 whitespace-nowrap text-sm font-semibold text-gray-900 border-gray-300 border-r"
                                                    >{employee.name}</td
                                                >
                                                <td
                                                    class="px-6 py-3 whitespace-nowrap text-sm font-semibold text-gray-900 border-gray-300 border-r"
                                                    >{employee.pin}</td
                                                >
                                                <td
                                                    class="px-6 py-3 whitespace-nowrap text-sm font-semibold text-gray-900 border-gray-300 border-r"
                                                    >{employee.hoursWorked}</td
                                                >
                                            </tr>
                                        {/each}
                                    {/if}
                                </tbody>
                            </table>
                        </div>

                        <!-- Pagination -->
                        <!-- {#if filteredEmployees.length > itemsPerPage} -->
                        <div
                            class="flex flex-col sm:flex-row items-center justify-between pt-4 border-t border-gray-200"
                        >
                            <div
                                class="text-base font-semibold text-gray-700 mb-2 sm:mb-0"
                            >
                                Showing <span class="font-medium"
                                    >{(currentPage - 1) * itemsPerPage +
                                        1}</span
                                >
                                to
                                <span class="font-medium"
                                    >{Math.min(
                                        currentPage * itemsPerPage,
                                        filteredEmployees.length,
                                    )}</span
                                >
                                of
                                <span class="font-medium"
                                    >{filteredEmployees.length}</span
                                > results
                            </div>

                            <div class="flex space-x-1">
                                <button
                                    on:click={() =>
                                        (currentPage = Math.max(
                                            1,
                                            currentPage - 1,
                                        ))}
                                    disabled={currentPage === 1}
                                    class="px-3 py-1 rounded-md text-base font-semibold text-gray-500 disabled:opacity-50"
                                >
                                    Previous
                                </button>

                                {#each Array(totalPages).fill(0) as _, i}
                                    {#if i + 1 === currentPage || i + 1 === currentPage - 1 || i + 1 === currentPage + 1 || i === 0 || i === totalPages - 1}
                                        <button
                                            on:click={() =>
                                                (currentPage = i + 1)}
                                            class={`px-3 py-1 border text-sm font-medium ${
                                                currentPage === i + 1
                                                    ? "bg-gray-200 border-[#02066F] cursor-pointer focus:outline-none focus:ring-1 focus:ring-[#02066F] "
                                                    : "bg-white border-gray-300 text-gray-700 hover:bg-gray-50"
                                            } rounded-sm`}
                                        >
                                            {i + 1}
                                        </button>
                                    {:else if i + 1 === currentPage - 2 || i + 1 === currentPage + 2}
                                        <span class="px-3 py-1 text-gray-700"
                                            >...</span
                                        >
                                    {/if}
                                {/each}

                                <button
                                    on:click={() =>
                                        (currentPage = Math.min(
                                            totalPages,
                                            currentPage + 1,
                                        ))}
                                    disabled={currentPage === totalPages}
                                    class="px-3 py-1 rounded-md text-base font-semibold text-gray-500 disabled:opacity-50"
                                >
                                    Next
                                </button>
                            </div>
                        </div>
                        <!-- {/if} -->
                        <!-- {/if} -->
                    </div>
                </div>
            </div>
        </main>
    </div>
</div>
