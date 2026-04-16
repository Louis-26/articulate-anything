articulate_anything/utils/prompt_utils.py

```python
# line 169-194
def _format_content(self, prompt_parts):
    """Format content for OpenAI API, handling both text and images"""
    if not isinstance(prompt_parts, list):
        # return prompt_parts                           # original
        return [{"type": "text", "text": prompt_parts}] # new
    formatted_content = []
    
    for part in prompt_parts:
        if isinstance(part, str):
            # formatted_content.append(part)  # original
            formatted_content.append({"type": "text", "text": part}) # new
        elif isinstance(part, Image.Image):
            # Convert PIL Image to base64 and format for OpenAI
            base64_image = self._encode_image_to_base64(part)
            formatted_content.append({
                "type": "image_url",
                "image_url": {
                    "url": f"data:image/jpeg;base64,{base64_image}",
                    "detail": "low"
                }
            })
        elif isinstance(part, dict) and part.get("type") == "image_url":
            # If it's already in the correct format, pass it through
            formatted_content.append(part)
    
    return formatted_content
```