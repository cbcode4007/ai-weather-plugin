## AI Weather Plugin

A small command-line program that delivers accurate weather information from Environment Canada upon request, tailored to the exact contents of the question. It is mainly meant to be used as part of the AI Operator framework, which will call it to defer any weather-related queries.

It takes the following parameters (besides the always necessary script name):
- User query string ("What's the temperature?", etc.)
- Region code (Canada-specific due to API, find your city in https://github.com/lschuyler/xbar-envcan-weather/blob/master/README.md#ontario)