
# 📦 Glance × n8n Plugin – Disk Space Monitor


---

## 📂 Disk space

1. Download the [`sonarr-diskspace.json`](https://github.com/whoissava/Glance-X-n8n/blob/main/sonarr-diskspace.json) file
2. In your n8n dashboard, go to Workflows → Import From File
3. Upload the downloaded JSON file
4. Configure your Sonarr credentials in the HTTP Request node
5. Add .Yaml Config to Glance 


<details>
  <summary><strong>▶ YAML Configuration for Glance</strong></summary>
  
  ```yaml
  - type: custom-api
    title: Disk Usage
    cache: 5m
    url: YOUR_N8N_WEBHOOK_URL
    template: |
      <ul class="list list-gap-10 collapsible-container" data-collapse-after="5">
      {{- range .JSON.Array "" }}
        {{- $path  := .String "path" }}
        {{- $label := .String "label" }}
        {{- $free  := .Float "freeSpaceGB" }}
        {{- $total := .Float "totalSpaceGB" }}
        {{- $used  := sub $total $free }}
        {{- $pct   := mul (div $used $total) 100 }}
        <li>
          <div class="size-h4 color-highlight block">
            {{- if gt (len $label) 0 }}{{ $label }}
            {{- else if eq $path "/" }}Root (/)
            {{- else }}{{ $path }}{{- end }}
          </div>
  
          <!-- Usage bar -->
          <div style="width:100%; background:#ddd; border-radius:4px; height:8px; margin:4px 0; overflow:hidden;">
            <div style="width:{{ printf "%.0f" $pct }}%; background:#69A794; height:100%;"></div>
          </div>
  
          <ul class="list-horizontal-text">
            <li>Free: <span class="color-green">{{ .String "freeSpaceGB" }} GB</span></li>
            <li>Total: {{ .String "totalSpaceGB" }} GB</li>
            <li class="color-orange">{{ printf "%.0f" $pct }}% used</li>
          </ul>
        </li>
      {{- end }}
      </ul>
  ```
</details>

Built by Sava, for the community.
