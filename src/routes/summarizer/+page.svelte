<script lang="ts">
    import "../../app.css";

    import {Sun, Moon, ChevronUp, Upload, Loader2} from "@lucide/svelte";
    import { Button } from "$lib/components/ui/button/index.js";
    import { Input } from "$lib/components/ui/input/index.js";
    import { Label } from "$lib/components/ui/label/index.js";
    import { Textarea } from "$lib/components/ui/textarea/index.js";
    import * as Drawer from "$lib/components/ui/drawer/index.js";

    import { readTextFile, readPdfFile, readDocxFile } from "$lib/utils";

    /**
     * Uploaded file
     */
    let fileInput: File;

    /**
     * Text to be summarized by the backend.
     */
    let text: string = "";

    /**
     * Additional Parameters from the user. The Large Language Model will take these into account when generating responses.
     */
    let additionalParams: string = "";

    /**
     * Temperature parameter for LLM responses.
     */
    let llmTemp: number = 0.7;

    /**
     * Number of paragraphs for summary.
     */
    let paragraphs: number = 1;

    /**
     * Number of sentences per paragraph.
     */
    let sentences: number = 5;

    /**
     * Response from LLM to be displayed
     */
    let summaryResponse: string = "";

    /**
     * flag for if LLM is processing text
     */
    let isLoading: boolean = false;

    /**
     * Send details to backend
     */
    function send() {
        const formData = new FormData();
        formData.append("text", text);
        formData.append("additionalParams", additionalParams);
        formData.append("temperature", llmTemp.toString());
        formData.append("paragraphs", paragraphs.toString());
        formData.append("sentences", sentences.toString());

        const formDataObject = Object.fromEntries(formData.entries());
        console.log("FormData:", formDataObject);
        summaryResponse = "";

        isLoading = true;
        setTimeout(() => {
            // TODO set fetching logic from backend
            summaryResponse = "This is the generated summary!";
            isLoading = false;
        }, 2000);
    }

    /**
     * Ensure that temperature is in range and round it off to the nearest hundredths.
     */
    function validateTemp(): void {
        console.log("e")
        llmTemp = Math.max(0, Math.min(2, llmTemp)); // Clamp value between 0 and 2
        llmTemp = Math.round(llmTemp * 100) / 100; // Round to nearest tenth
    }

    /**
     * Get label for temperature UI
     * @param temp
     */
    function getTemperatureLabel(temp: number): string {
        if (temp >= 0.0 && temp <= 0.3) return "Objective";
        if (temp > 0.3 && temp <= 0.7) return "Balanced";
        if (temp > 0.7 && temp <= 1.2) return "Creative";
        return "Chaotic";
    }

    /**
     * Get color for temperature UI
     * @param temp
     */
    function getTempColor(temp: number) {
        if (temp <= 0.3) return "#3b82f6"; // Green (Objective)
        if (temp <= 0.7) return "#22c55e"; // Blue (Balanced)
        if (temp <= 1.2) return "#eab308"; // Yellow (Creative)
        return "#f97316"; // Red (Chaotic)
    }

    /**
     * Open file upload window
     */
    function triggerFileUpload() {
        const fileInput = document.getElementById("text-file");
        if (fileInput) {
            fileInput.click();
        } else {
            console.error("Element with ID 'text-file' not found.");
        }
    }

    /**
     * Upload file and extract text
     * @param event
     */
    async function handleFileUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        if (!input.files || input.files.length === 0) return;

        const file = input.files[0];
        const fileType = file.name.split(".").pop()?.toLowerCase();

        if (fileType === "txt") {
            text = await readTextFile(file);
        } else if (fileType === "pdf") {
            text = await readPdfFile(file);
        } else if (fileType === "docx") {
            text = await readDocxFile(file);
        } else {
            console.error("Unsupported file type");
        }

        fileInput = null;
    }
</script>

<div class="flex justify-center items-start gap-25 w-full">

    <!--  User Input (File Upload/Text Edit  -->
    <div class="w-[90%] md:w-3/7">
        <div class="grid w-1/2 max-w-sm items-center gap-1.5 mb-4">
            <Label for="picture">Enter Text to be Summarized OR</Label>
            <Button variant="default" type="button" class="flex items-center gap-2" onclick={triggerFileUpload}>
                <Upload class="w-5 h-5" />
                <span>Upload File</span>
            </Button>
        </div>
        <Input id="text-file" type="file" accept=".docx,.pdf,.txt" class="hidden" bind:value={fileInput} onchange={handleFileUpload} />
        <Textarea
                bind:value={text}
                class="resize-none w-full h-[calc(100vh-18rem)] p-5"
                placeholder="Type your message here."
                id="message"
        />
    </div>

    <div class="hidden md:block w-[90%] md:w-3/7">
        <div class="flex items-center gap-2 mb-2">
            <div class="text-lg font-semibold">Summary</div>
        </div>
        <div class="hidden md:block response-box w-full h-[calc(100vh-15.5rem)] border border-border rounded-md overflow-auto p-5">
            {#if isLoading}
                <span class="animate-pulse text-muted-foreground">Loading summary...</span>
            {:else}
                {summaryResponse || "Your summary will appear here..."}
            {/if}
        </div>
    </div>
</div>


<div class="fixed bottom-0 left-0 w-full flex items-center gap-2 p-2 bg-card">
    <div class="hidden md:block">
        <Drawer.Root>
            <div class="fixed bottom-[4.7rem] left-0 w-full flex justify-center">
                <Drawer.Trigger>
                    <Button variant="outline" class="w-100 h-7 flex items-center justify-center">
                        <ChevronUp class="w-5 h-5" />{getTemperatureLabel(llmTemp)} ({llmTemp}) | {paragraphs} Paragraphs | {sentences} Sentences<ChevronUp class="w-5 h-5" />
                    </Button>
                </Drawer.Trigger>
            </div>
            <Drawer.Content class="drawer-content pointer-events-auto bg-primary p-4">
                <div class="grid grid-cols-5 gap-4">
                    <!-- LLM Temperature Slider (3 Columns) -->
                    <div class="col-span-4 flex flex-col gap-2">
                        <label for="llm-temp" class="text-sm font-medium">How would you like your responses?</label>
                        <div class="flex flex-col items-center w-full">
                            <div class="flex justify-between w-full px-2">
                                <span class="font-bold text-sm text-blue-500">Objective</span>
                                <span class="absolute left-[21%] font-bold text-sm text-green-500">Balanced</span>
                                <span class="absolute left-[38.5%] font-bold text-sm text-yellow-500">Creative</span>
                                <span class="text-sm font-bold text-orange-500">Chaotic</span>
                            </div>
                            <Input
                                    type="range"
                                    id="llm-temp"
                                    min="0"
                                    max="2"
                                    step="0.01"
                                    bind:value={llmTemp}
                                    class="w-full mt-1"
                                    style="accent-color: {getTempColor(llmTemp)};"
                            />
                        </div>


                        <!-- Number Input Below the Slider -->
                        <Input
                                type="number"
                                id="llm-temp"
                                bind:value={llmTemp}
                                min="0"
                                max="2"
                                step="0.01"
                                class="w-full border rounded-md px-2 py-1 text-center"
                                onblur={validateTemp}
                        />
                    </div>

                    <!-- Paragraphs & Sentences Inputs -->
                    <div class="col-span-1 flex flex-col gap-2">
                        <label for="paragraphs" class="text-sm font-medium">Paragraphs</label>
                        <Input
                            type="number"
                            id="paragraphs"
                            bind:value={paragraphs}
                            min="1"
                            class="border rounded-md px-2 py-1"
                        />

                        <label for="sentences" class="text-sm font-medium">Sentences per Paragraph</label>
                        <Input
                            type="number"
                            id="sentences"
                            bind:value={sentences}
                            min="1"
                            class="border rounded-md px-2 py-1"
                        />
                    </div>
                </div>
                <Drawer.Footer>
                </Drawer.Footer>
            </Drawer.Content>
        </Drawer.Root>
    </div>
    <Textarea
            bind:value={additionalParams}
            class="flex-1 md:h-14 h-20 resize-none bg-card text-foreground border-border outline-none focus:outline-ring"
            placeholder="Enter additional requests for response (optional)..."
    />
    <Button variant="default" onclick={send} class="flex items-center gap-2" disabled={isLoading}>
        {#if isLoading}
            <Loader2 class="w-5 h-5 animate-spin" />
        {:else}
            Send
        {/if}
    </Button>
</div>