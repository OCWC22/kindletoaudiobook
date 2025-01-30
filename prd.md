# Product Requirements Document: AI Kindle Audiobook Converter

## 1. Introduction

This document outlines the requirements for an MVP (Minimum Viable Product) AI-powered coding assistant designed to convert Kindle eBooks into audiobooks, along with summaries and key takeaways. The goal is to provide a simple, reliable solution without over-engineering.

## 2. Goals

*   **Primary Goal:** To enable users to convert Kindle eBooks into high-quality audiobooks.
*   **Secondary Goals:**
    *   Provide chapter-wise summaries and key takeaways.
    *   Generate overall book summary and key takeaways.
    *   Offer a user-friendly experience via a web interface.
*   **Success Metrics:**
    *   Number of successful eBook-to-audiobook conversions.
    *   User engagement with summary and key takeaway features.
    *   Positive user feedback and adoption rate.

## 3. Target User

*   Users who prefer listening to reading.
*   Individuals with busy schedules who want to consume books while multitasking.
*   Users seeking tools for more accessible content consumption.

## 4. User Stories

*   As a user, I want to easily upload my Kindle eBooks.
*   As a user, I want the system to automatically convert the text to speech.
*   As a user, I want to listen to my audiobooks directly in the browser.
*   As a user, I want to see a summary and key takeaways for the entire book.
*   As a user, I want to see a summary and key takeaways for every chapter.

## 5. Functional Requirements

*   **eBook Upload:**
    *   Users can upload Kindle eBooks (ideally .azw3 or .mobi format) via a web interface.
    *   The system must validate the file format before proceeding.
*   **Browser Automation:**
    *   The system uses `browser-use` ([https://github.com/browser-use/browser-use](https://github.com/browser-use/browser-use)) to simulate user interaction with the Kindle Cloud Reader in the browser.
    *   The agent should navigate to the correct book in the user's library.
    *   The agent will use the provided library to scrape the textual content of the book, page by page or chapter by chapter.
*   **Text Extraction:**
    *   The system must extract text content from the opened Kindle Book in the browser.
    *   The system should handle pagination and other layout complexities with the `browser-use` tool.
*   **Text Summarization and Key Takeaways:**
    *   The system uses the selected LLM (`Deepseek V3/R1` or `Gemini 2.0 Flash`) to:
        *   Generate chapter-wise summaries and key takeaways.
        *   Generate an overall book summary and key takeaways.
    *   Summaries must be concise and informative.
*   **Text-to-Speech:**
    *   The system utilizes the Eleven Labs API for generating audio from the extracted and summarized text.
    *   Audio should be high-quality and natural-sounding.
    *   The audio should be chunked by chapter.
*   **Audio Playback:**
    *   Users can listen to the generated audiobooks via a built-in audio player.
    *   Playback controls (play, pause, skip, etc.) should be available.
*   **User Interface:**
    *   A simple web interface to allow users to upload, process the ebook, and listen to their audiobooks and view their summaries and key takeways.

## 6. Non-Functional Requirements

*   **Reliability:** The system must reliably handle a diverse range of Kindle book formats, or return an error if it cannot. The system must always provide clear feedback to the user.
*   **Performance:** The system must process books in a reasonable amount of time, without excessive delays.
*   **Scalability:** The system should be designed to handle a moderate increase in user load.
*   **Security:** User data and uploaded files must be handled securely.
*   **Maintainability:** The codebase should be well-documented and easy to maintain.

## 7. Technical Design

*   **Architecture:**
    *   **Backend (FastAPI):**
        *   API endpoints for file upload, processing, text summarization, and audio generation.
        *   Handles asynchronous tasks using Celery or similar for long-running processes.
    *   **Browser Automation:** Uses `browser-use` library for controlling the web browser, navigating the Kindle UI, and extracting text content.
    *   **LLM Integration:** API calls to either Deepseek V3/R1 or Gemini 2.0 Flash for generating summaries and key takeaways.
    *   **Text-to-Speech:** API calls to Eleven Labs for converting text to audio.
    *   **Frontend (HTML/JS):**
        *   Basic web interface for file upload and audio playback.
        *   Displays summaries and key takeaways
*   **Technology Stack:**
    *   **Programming Language:** Python 3.10 or higher
    *   **Web Framework:** FastAPI
    *   **Browser Automation Library:** `browser-use`
        *   Documentation: [https://github.com/browser-use/browser-use](https://github.com/browser-use/browser-use)
    *   **LLM:** Deepseek V3/R1 or Gemini 2.0 Flash (via API)
        *   Deepseek documentation: [https://deepseek.com/](https://deepseek.com/) (You'll need to explore their API documentation)
        *   Gemini documentation: [https://ai.google.dev/](https://ai.google.dev/) (see Gemini API documentation)
    *   **Text-to-Speech:** Eleven Labs API
        *   Documentation: [https://elevenlabs.io/docs](https://elevenlabs.io/docs)
    *   **Asynchronous Task Queue:** Celery or similar (Optional, can be avoided for MVP)
    *   **Frontend:** Basic HTML, CSS, JavaScript

*   **Data Flow**
    1. User uploads Kindle ebook via the frontend
    2. Backend API receives the uploaded file.
    3. Backend initiates `browser-use`, launching a browser instance and navigating to Kindle Cloud Reader, simulating user login and book navigation.
    4. Using `browser-use` to read content page by page or chapter by chapter.
    5. Extracted text is sent to the chosen LLM (Deepseek or Gemini) for summarization.
    6. LLM generates chapter-wise and full book summaries/key takeaways.
    7. Text is converted to audio using Eleven Labs API.
    8. Audio and summaries are saved and are made available via the frontend.
    9. User can listen to the audiobook and view summaries in the frontend.

*   **Specific Technical Details:**
    *   **Browser-use Implementation:**
        *   Use `browser_use.Browser` class to control browser interactions.

            ```python
            # Example code using browser-use
            from browser_use import Browser
            from playwright.sync_api import TimeoutError

            def navigate_to_kindle_book(book_url, browser_instance):

                try:
                  browser_instance.goto(book_url)

                  # Wait for the book to load (adjust selectors and timeouts as needed)
                  browser_instance.wait_for_selector(".book-content", timeout=10000)

                except TimeoutError as e:
                    print(f"Error loading book: {e}")
                    return None
                return browser_instance

            def extract_text_from_kindle(browser_instance):
                # Example code for text extraction, adapt this to work with the DOM
                # you will have to fine tune selectors based on the browser
                all_text = ""
                while True:
                    try:
                       # wait for the content
                        browser_instance.wait_for_selector(".kp-page-content", timeout=5000)

                       #Extract all text from content
                        page_text = browser_instance.query_selector(".kp-page-content").inner_text()

                       # Append extracted text
                        all_text += page_text + "\n"

                       # Check if there is a next page button
                        next_page_button = browser_instance.query_selector(".nextPage")

                       # If there is no next page, exit
                        if not next_page_button:
                            break

                        #Move to next page
                        next_page_button.click()

                    except TimeoutError:
                        print("Timed out waiting for selector")
                        break
                    except Exception as e:
                        print(f"Error encountered while scraping: {e}")
                        break

                return all_text

             def process_kindle_book(book_url):
                # setup a browser object
                browser = Browser()
                #Start the browser, passing in the browser context object
                with browser.start() as browser_instance:
                    #Go to the book page
                  browser_instance = navigate_to_kindle_book(book_url, browser_instance)
                    if not browser_instance:
                        print("Could not navigate to the book")
                        return None

                    #extract the text from the book
                    book_text = extract_text_from_kindle(browser_instance)
                    if not book_text:
                        print("Could not scrape text from book")
                        return None
                    #process
                    return book_text
            ```

        *   Implement specific navigation steps to get to the user library and desired book. (This will be based on the Kindle UI structure and requires careful selector creation)
        *   Use the correct selectors to scrape the text. (Use the browser's built in inspect tool to find the right selectors)

    *   **LLM Integration:**
        *   Use the respective API client libraries for Deepseek or Gemini.

            ```python
            # Hypothetical example using an LLM client
            import os

            #Dummy Class to simulate the behavior of either client
            class LLMClient:
              def __init__(self, api_key, model_name):
                 self.api_key = api_key
                 self.model_name = model_name

              def summarize_text(self, text_content, prompt):
                   # In real use cases, this makes a call to the LLM API
                   # for the sake of this example, it will just return a dummy text
                   return f"Summary for text using {self.model_name}: {text_content[:50]} ... "

            # Example code for calling the LLM for summarization
            def generate_summary(text, model_type):
                api_key = os.getenv("LLM_API_KEY")

                if not api_key:
                   print("No api key provided")
                   return None

                if model_type == "deepseek":
                  llm_client = LLMClient(api_key, "deepseek-v3")
                elif model_type == "gemini":
                  llm_client = LLMClient(api_key, "gemini-2.0-flash")
                else:
                  print("invalid model")
                  return None

                prompt = "Summarize this text in a concise manner and provide key takeaways:" # you can further extend this
                summary = llm_client.summarize_text(text, prompt)
                return summary
            ```

        *   Construct the proper prompt for summarization tasks. (See above example)
        *   Handle API rate limits and errors.

    *   **Eleven Labs Implementation:**

        ```python
        # Example using Eleven Labs API (simplified for illustrative purposes)
        import os
        import requests

        ELEVEN_LABS_API_KEY = os.getenv("ELEVEN_LABS_API_KEY")
        VOICE_ID = "pNInz6obpgDzWyq0mJkO" #example voice

        def text_to_speech(text, api_key):
           url = "https://api.elevenlabs.io/v1/text-to-speech/pNInz6obpgDzWyq0mJkO"
           headers = {
             "xi-api-key": api_key,
             "Content-Type": "application/json"
            }
           data = {
            "text": text,
            "voice_settings": {
               "stability": 0.5,
               "similarity_boost": 0.5
             }
            }

           try:
              response = requests.post(url, headers=headers, json=data, stream=True)
              response.raise_for_status()  # Raise HTTPError for bad responses (4xx or 5xx)

              if response.status_code == 200:
                with open("output.mp3", "wb") as f:
                  for chunk in response.iter_content(chunk_size=1024):
                       f.write(chunk)
                return "output.mp3"  #return the name of the output audio file
              else:
                print(f"Error generating audio: {response.status_code}, {response.text}")
                return None
           except requests.exceptions.RequestException as e:
              print(f"An error occurred during the request: {e}")
              return None

        def process_audio(text_content):
            api_key = os.getenv("ELEVEN_LABS_API_KEY")
            if not api_key:
              print("Missing eleven labs api key")
              return None
            audio_file = text_to_speech(text_content, api_key)
            return audio_file
        ```

        *   Use the Eleven Labs API client library.
        *   Implement logic to split text into chunks to avoid the text limitations with Eleven Labs, and then concatenate the output audio.

*   **Error Handling:**
    *   Implement robust error handling throughout the system.
    *   Provide informative error messages to the user.
*   **API Design:**
    *   Use FastAPI's dependency injection and data validation.
    *   Implement proper authentication (if needed, can be excluded for MVP)

## 8. User Interface (UI) Sketch

*   **Page 1: Upload Page**
    *   "Upload Kindle eBook" button for file uploads.
    *   "Browse Files" with file dialog.
    *   Simple notification for file upload success or failure.
    *   "Start Conversion" button which starts the ebook processing on the backend

*   **Page 2: Processing Page**
    *   Simple loading indicator with a "Processing..." message.
    *   Estimated time or percentage progress (if possible).
    *   (Optionally) Log output or status messages.

*   **Page 3: Audiobook and Summary Page**
    *   **Audio Player:**
        *   Basic play/pause controls.
        *   (Optional) forward/back controls, progress bar, volume.
    *   **Chapter Summaries and Key Takeaways:**
        *   List of chapters.
        *   Expandable content for each chapter to show summaries/key takeaways.
    *   **Full Book Summary:**
        *   Section to display the full book summary and key takeaways.

## 9. MVP Scope

*   **Phase 1 (MVP):**
    *   Focus on core functionality: eBook to audio conversion, basic summarization, and audio playback.
    *   Basic UI: File upload, processing indicator, simple audio player, basic summary display.
    *   Support one LLM and one TTS provider.
    *   Minimal error handling and UI/UX polish.
    *   No complex features like user accounts or custom settings.
    *   Prioritize reliability over advanced features.

## 10. Future Enhancements (Out of Scope for MVP)

*   User accounts with saved audiobooks and history.
*   Choice of different LLM and TTS providers.
*   Custom voice selection.
*   More advanced summarization options.
*   Support for more ebook formats.
*   More comprehensive error handling.
*   Improved progress indicators.
*   Full featured UI/UX.
*   Mobile responsiveness.
*   User settings for customization.

## 11. Risk and Challenges

*   Handling complex layouts and extracting text correctly.
*   API rate limits for the LLM and Eleven Labs.
    *   Implement logic to handle rate limiting, potentially using retries with exponential backoff.
*   Managing the browser automation process consistently.
    *   Ensure that the browser environment is reliable and compatible with Kindle Cloud Reader to avoid unexpected errors.
*   Ensuring high-quality audio output.
    *   Tune the voice settings (stability, similarity boost) in Eleven Labs to optimize audio quality.
*   Handling large eBook files efficiently.
    *   Process the book in chunks or chapter wise to avoid memory issues and increase processing speed.

## 12. Conclusion

This PRD provides a detailed plan for the development of a simple and reliable AI Kindle Audiobook converter MVP. By focusing on core functionalities and clear objectives, this product will offer a high degree of value for its target users. This product aims to be a solid base that can be built upon for future product iterations.

## Important Notes:

*   **Placeholders:** The provided LLM and Eleven Labs code is basic. The actual implementations will require the usage of their specific SDKs/libraries and error handling for each.
*   **API Keys:** In the examples, the API keys are assumed to be in environment variables ( `LLM_API_KEY` and `ELEVEN_LABS_API_KEY`). Ensure that API keys are securely handled and not hardcoded.
*   **Error Handling:** Basic error handling is implemented but should be extended for production.
*   **Selector Tuning:**  The browser automation part with `browser-use` is very dependent on the Kindle webpage structure. The CSS selectors used will need to be updated based on the structure of the webpage.
*   **Further Development:** The code examples are for illustrative purposes and will need to be further developed to achieve full functionality. You will need to refer to the documentation for each of the used libraries to get them working. You should also look into using a dependency injection framework to avoid hardcoding the API keys as shown in the examples.