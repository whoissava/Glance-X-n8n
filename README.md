
# 📦 Glance × n8n Plugin

This directory collects a series of custom plugins for n8n, designed to extend the functionality of Glance.
---

## 📂 Disk space

![image](https://github.com/user-attachments/assets/b0aa3f1f-876a-4cc6-aa32-df32d9d0cb03)



1. Download the [`sonarr-diskspace.json`](https://github.com/whoissava/Glance-X-n8n/blob/main/sonarr-diskspace.json) file
2. In your n8n dashboard, go to Workflows → Import From File
3. Upload the downloaded JSON file
4. Configure your Sonarr credentials in the HTTP Request node
5. Add .Yaml Config to Glance 


<details>
  <summary><strong> YAML Configuration for Glance</strong></summary>
  
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


## 📂 Nextup sonarr / radarr

1. Download the [`sonarr-calendar.json`]([https://github.com/whoissava/Glance-X-n8n/blob/main/sonarr-diskspace.json](https://github.com/whoissava/Glance-X-n8n/blob/main/sonarr_calendar.json) and [`radarr-calendar.json`]([https://github.com/whoissava/Glance-X-n8n/blob/main/sonarr-diskspace.json](https://github.com/whoissava/Glance-X-n8n/blob/main/radarr_calendar.json)  file
2. In your n8n dashboard, go to Workflows → Import From File
3. Upload the downloaded JSON file
4. Configure your Sonarr credentials in the HTTP Request node
5. Add .Yaml Config to Glance 

<details>
  <summary><strong> YAML Configuration for Glance</strong></summary>
  
  ```yaml
      - type: group    #voglio del sushi   
        widgets:
        - type: custom-api
          title: Radarr – Coming soon
          cache: 30m
          url: N8N webhook
          headers: {}
          template: |
            <ul class="list list-gap-10">
              {{- $items := .JSON.Array "" }}
              {{- if eq (len $items) 0 }}
                <li><p class="size-h5 color-paragraph">Nessuna uscita prevista</p></li>
              {{- else }}
                {{- range $items }}
                  {{- $title := .String "title" }}
                  {{- $date  := .String "physicalRelease" }}
                  {{- if eq $date "" }}
                    {{- $date = .String "digitalRelease" }}
                  {{- end }}
                  {{- if eq $date "" }}
                    {{- $date = .String "releaseDate" }}
                  {{- end }}
                  {{- $img := "" }}
                  {{- $imgs := .Array "images" }}
                  {{- range $imgs }}
                    {{- if eq (.String "coverType") "poster" }}
                      {{- $img = .String "remoteUrl" }}
                    {{- end }}
                  {{- end }}
                  <li class="flex items-center gap-4">
                    {{- if $img }}
                      <img src="{{ $img }}"
                          alt="Locandina {{ $title }}"
                          style="width:50px; border-radius:4px;">
                    {{- else }}
                      <div style="width:50px; height:75px; background:#444; border-radius:4px;"></div>
                    {{- end }}
                    <div class="grow min-width-0">
                      <div class="size-h5 color-highlight block text-truncate">{{ $title }}</div>
                      <ul class="list-horizontal-text">
                        <li>Uscita: {{ slice $date 0 10 }}</li>
                      </ul>
                    </div>
                  </li>
                {{- end }}
              {{- end }}
            </ul>



        - type: custom-api
          title: Sonarr – Prossime Uscite
          cache: 30m
          url: N8N Webhook 
          headers: {}
          template: |
            <ul class="list list-gap-10">
              {{- $items := .JSON.Array "" }}
              {{- if eq (len $items) 0 }}
                <li><p class="size-h5 color-paragraph">Nessuna uscita prevista</p></li>
              {{- else }}
                {{- range $items }}
                  {{- /* Se è un array interno, iteralo; altrimenti usa direttamente questo elemento */}}
                  {{- $nested := .Array "" }}
                  {{- if gt (len $nested) 0 }}
                    {{- range $nested }}
                      {{- /* === inizio rendering episodio === */}}
                      {{- $series := .String "series.title" }}
                      {{- $img    := .String "series.images.1.remoteUrl" }}
                      {{- $season := .Int    "seasonNumber" }}
                      {{- $ep     := .Int    "episodeNumber" }}
                      {{- $raw    := printf "%v" (.Get "airDateUtc") }}
                      {{- /* safe‑slice della data */}}
                      {{- $date := "Data non disponibile" }}
                      {{- if ge (len $raw) 10 }}
                        {{- $date = slice $raw 0 10 }}
                      {{- end }}
                      {{- /* mostra solo se S>0 e E>0 */}}
                      {{- if and (gt $season 0) (gt $ep 0) }}
                        <li class="flex items-center gap-4">
                          {{- if $img }}
                            <img src="{{ $img }}"
                                alt="Cover {{ $series }}"
                                style="width:50px; border-radius:4px;">
                          {{- else }}
                            <div style="width:50px; height:75px; background:#444; border-radius:4px;"></div>
                          {{- end }}
                          <div class="grow min-width-0">
                            <div class="size-h5 color-highlight block text-truncate">{{ $series }}</div>
                            <ul class="list-horizontal-text">
                              <li>S{{ printf "%02d" $season }}E{{ printf "%02d" $ep }}</li>
                              <li>{{ $date }}</li>
                            </ul>
                          </div>
                        </li>
                      {{- end }}
                      {{- /* === fine rendering episodio === */}}
                    {{- end }}
                  {{- else }}
                    {{- /* rendering diretto per elemento non‑annidato */}}
                    {{- $series := .String "series.title" }}
                    {{- $img    := .String "series.images.0.remoteUrl" }}
                    {{- $season := .Int    "seasonNumber" }}
                    {{- $ep     := .Int    "episodeNumber" }}
                    {{- $raw    := printf "%v" (.Get "airDateUtc") }}
                    {{- $date := "Data non disponibile" }}
                    {{- if ge (len $raw) 10 }}
                      {{- $date = slice $raw 0 10 }}
                    {{- end }}
                    {{- if and (gt $season 0) (gt $ep 0) }}
                      <li class="flex items-center gap-4">
                        {{- if $img }}
                          <img src="{{ $img }}"
                              alt="Cover {{ $series }}"
                              style="width:50px; border-radius:4px;">
                        {{- else }}
                          <div style="width:50px; height:75px; background:#444; border-radius:4px;"></div>
                        {{- end }}
                        <div class="grow min-width-0">
                          <div class="size-h5 color-highlight block text-truncate">{{ $series }}</div>
                          <ul class="list-horizontal-text">
                            <li>S{{ printf "%02d" $season }}E{{ printf "%02d" $ep }}</li>
                            <li>{{ $date }}</li>
                          </ul>
                        </div>
                      </li>
                    {{- end }}
                  {{- end }}
                {{- end }}
              {{- end }}
            </ul>

  ```
</details>

Built by Sava, for the community.
