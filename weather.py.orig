# © 2026 Colin Bond
# All rights reserved.

import os
import logging
import re
import sys
import requests
from datetime import datetime
from ailib import Payload
from preferences import Preferences

class Weather:
    """
    Provides functions to get Environment Canada weather API and query it with AI for relevant information.
    Version 0.0.3:    
        - added extra command parameter for location code within Canada (eg. on-117 = Oshawa), and fixed debug log setting issues
    Version 0.0.2:
        - simple elapsed time calculation and display for both the API call and AI call each
    """

    version = "0.0.3"

    def __init__(self, preferences_file: str):
        # Instantiate Settings/Prefs
        self.preferences = Preferences(preferences_file)

        # Load up settings/preferences and then instantiate Payload with preferred config
        prompts_file = self.preferences.get_setting_val("Prompts File")
        chat_hist_file = self.preferences.get_setting_val("Chat History File")
        self.log_file = self.preferences.get_setting_val("Log File")
        log_mode = self.preferences.get_setting_val("Log Mode")

        # Setup Log file and current mode (Debug or Info)
        self._configure_logging(log_mode)

        # Load Open AI Key from environment variable, or settings/prefs for backward compatability
        api_key = self._load_openai_key()
        if api_key == "":
            api_key = self.preferences.get_setting_val("OpenAI Key")                

        # Instantiate Payload with settings values retrieved
        self.payload = Payload(prompts_file, chat_hist_file, api_key)

        # Flags to ensure smooth and uninterrupted weather fetching
        self._fetching = False
        self.loading = True
        self.fetch_error = None
        self.data = None

    def _load_openai_key(self):
        # Get key from environment variables
        api_key = os.getenv("OPENAI_API_KEY")

        if api_key is None:
            logging.error("OPENAI_API_KEY not found in environment variables")
            return ""

        else:
            return api_key

    def _configure_logging(self, log_mode: str):
        """Debug for DEBUG mode, Anything else for INFO"""
        # Default logging level set to Info but override to DEBUG with parameter
        log_level = logging.INFO        
        if log_mode.lower() == "debug":
            log_level = logging.DEBUG

        logging.basicConfig(
            filename=self.log_file,
            level=log_level,
            format='%(asctime)s - %(levelname)s - %(message)s'
        )    

    def fetch_weather(self, region_code: str):
        if self._fetching:
            return
        self._fetching = True
        api = "https://api.weather.gc.ca/collections/citypageweather-realtime/items/{}?f=json"
        api = api.format(region_code)
        try:
            resp = requests.get(
                api,
                timeout=10
            )
            if resp.status_code == 200:
                decoded = resp.json()
                if "properties" in decoded:
                    self.data = decoded["properties"]
                    return self.data
        except Exception as e:
            self.fetch_error = str(e)
        finally:
            self._fetching = False
            self.loading = False

    def _clean_ai_response(self, ai_response: str):
        # --- Clean AI response (remove ```json or ``` etc.) ---
        cleaned = re.sub(r'^```[a-zA-Z]*\n?', '', ai_response.strip())
        cleaned = re.sub(r'\n?```$', '', cleaned.strip())

        response_text = cleaned
        return response_text
  
    def process_ai_response(self, ai_response):

        clean_response = self._clean_ai_response(ai_response)

        response_text = clean_response
        return response_text

# ============================================================= M A I N ==================================================

    def main(self):

        if len(sys.argv) < 3:
            print("Usage: python news.py <MESSAGE_TO_AI> <REGION_CODE>, <LOG_MODE> is Optional 'Debug' or 'Info' (default)")
            return("<MESSAGE_TO_AI> and <REGION_CODE> parameters are required, Weather exiting")
            sys.exit(1)
        
        script = sys.argv[0]
        user_msg = sys.argv[1]
        region_code = sys.argv[2]

        # Indicate start of the log for this run of the program
        logging.info("💡")
        logging.info(f"AI Backup Analyzer v{self.version} class currently using ModelConnection v{self.payload.connection.version}, PromptBuilder v{self.payload.prompts.version}, ChatHistoryManager v{self.payload.history.version}, Preferences v{self.preferences.version}, Payload v{self.payload.version}")      

        # Initialize AI analyzer, loading system instructions, token capacity, model, verbosity and reasoning effort for the response
        prompt = "weather"
        self.payload.prompts.load_prompt(prompt)
        self.payload.connection.set_maximum_tokens(500)
        
        # GPT 5 mini
        # self.payload.connection.set_model("gpt-5-mini")        
        # self.payload.connection.set_verbosity("low")
        # self.payload.connection.set_reasoning_effort("minimal")

        # GPT 4o mini
        # self.payload.connection.set_model("gpt-4o-mini")        

        # GPT 5 nano
        self.payload.connection.set_model("gpt-5-nano")        
        self.payload.connection.set_verbosity("low")
        self.payload.connection.set_reasoning_effort("minimal")

        # Set string of weather data and user query in AI prompt
        startTime = datetime.now()
        weather = self.fetch_weather(region_code)
        endTime = datetime.now()
        elapsedTime = (endTime - startTime).total_seconds()
        logging.info(f"Weather API elapsed seconds: {elapsedTime}")
        prompt_addendum = f"Weather: {weather}"  

        # Prepare and send message to AI
        logging.info(f"Model: {self.payload.connection.model}, Verbosity: {self.payload.connection.verbosity}, Reasoning: {self.payload.connection.reasoning_effort}, Max Tokens: {self.payload.connection.maximum_tokens}")
        # logging.debug(f"Payload \n\nSystem Prompt: {self.payload.prompts.get_prompt()} \n\nSpecial/Dynamic Content: \n{weather} \n\nUser Message: {user_message}")
        logging.debug(f"Payload \n\nSystem Prompt: {self.payload.prompts.get_prompt()} \n\nSpecial/Dynamic Content: \n{weather} \n\nUser Message: {user_msg}")
        # Keep history for context
        self.payload.Auto_Add_AI_Response_To_History = True

        # Send the user mesage to AI and receive the response
        startTime = datetime.now()
        reply = self.payload.send_message(user_msg, prompt, prompt_addendum)
        endTime = datetime.now()
        elapsedTime = (endTime - startTime).total_seconds()
        logging.info(f"AI elapsed seconds: {elapsedTime}")
        logging.info(f"Assistant Raw Reply: {reply}")

        ret = self.process_ai_response(reply)
        if ret:
            response_val = ret or ""
            logging.debug(f"process_ai_response Return Value: {ret}")
        else:
            response_val = "AI response error, could not clean"
            
        logging.info(f"main() Returning (response_val): [{response_val}]\n")        

        return response_val     

if __name__ == "__main__":
    # Change the current working directory to this script for imports and relative pathing
    program_path = os.path.dirname(os.path.abspath(__file__))
    os.chdir(program_path)

    wrapper = Weather("weather.json")

    response_output = wrapper.main()
  
    print(f"{response_output}")
    
    
