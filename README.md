
# 📦 Glance × n8n Plugin

This directory collects a series of custom plugins for n8n, designed to extend the functionality of Glance.

If you need plugins without N8N, check [`this`](https://github.com/whoissava/Glance-Plugin) 

Remember to put your data like ip or credentials to the http request 
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

1. Download the
[`sonarr_calendar.json`](https://github.com/whoissava/Glance-X-n8n/blob/main/sonarr-diskspace.json) and [`radarr_calendar.json`](https://github.com/whoissava/Glance-X-n8n/blob/main/sonarr-diskspace.json)  file
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
                    <div class="grow min-width-0" style="margin-left: 1.5rem;">
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
          title: Sonarr – Coming soon
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
                        <div class="grow min-width-0" style="margin-left: 1.5rem;">
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

## 📂 qBittorrent Tracker

![image](https://github.com/user-attachments/assets/35cab417-be3c-4533-ba15-8470b5a0406e)


1. Download the
[`qbittorrent.json`](https://github.com/whoissava/Glance-X-n8n/blob/main/qbittorrent.json)file
2. In your n8n dashboard, go to Workflows → Import From File
3. Upload the downloaded JSON file
4. Configure your ip in the HTTP Request node
5. Add .Yaml Config to Glance 


<details>
  <summary><strong> YAML Configuration for Glance</strong></summary>
  
  ```yaml
      - type: group    #aaaaaaaaa   
        widgets:
        - type: custom-api
          title: "Status"
          cache: 5m
          url: n8n webhook
          template: |
            <div style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; text-align: center;">
              
              <!-- BLOCCO 1: ATTIVI -->
              <div>
                <div style="font-size: 1.5rem; font-weight: bold;">{{ .JSON.Int "summary.downloading" }}</div>
                <div style="opacity: 0.7;">Attivi</div>
                <div style="display: flex; justify-content: space-around; font-size: 0.8rem; margin-top: 0.5rem;">
                  <div>
                    <div>{{ .JSON.Int "summary.paused" }}</div>
                    <div style="opacity: 0.6;">Pausa</div>
                  </div>
                  <div>
                    <div>{{ .JSON.Int "summary.stopped" }}</div>
                    <div style="opacity: 0.6;">Stop</div>
                  </div>
                </div>
              </div>

              <!-- BLOCCO 2: TOTALE -->
              <div>
                <div style="font-size: 1.5rem; font-weight: bold;">{{ .JSON.Int "summary.total" }}</div>
                <div style="opacity: 0.7;">Totale</div>
                <div style="display: flex; justify-content: space-around; font-size: 0.8rem; margin-top: 0.5rem;">
                  <div>
                    <div>{{ .JSON.Int "summary.seeding" }}</div>
                    <div style="opacity: 0.6;">Seed</div>
                  </div>
                  <div>
                    <div>{{ .JSON.Int "summary.completed" }}</div>
                    <div style="opacity: 0.6;">Completati</div>
                  </div>
                </div>
              </div>

              <!-- BLOCCO 3: ERRORI + ALTRI -->
              <div>
                <div style="font-size: 1.5rem; font-weight: bold;">{{ .JSON.Int "summary.errored" }}</div>
                <div style="opacity: 0.7;">Errori</div>
                <div style="display: flex; justify-content: center; font-size: 0.8rem; margin-top: 0.5rem;">
                  <div>
                    <div>{{ .JSON.Int "summary.others" }}</div>
                    <div style="opacity: 0.6;">Altri</div>
                  </div>
                </div>
              </div>

            </div>

            <!-- FOOTER: VELOCITÀ -->
            <div style="margin-top: 1rem; padding-top: 0.5rem; border-top: 1px solid rgba(255,255,255,0.1); text-align: center; font-size: 0.9rem; opacity: 0.8;">
              ↓ Download: <strong>{{ .JSON.String "summary.totalDownloadSpeed" }}</strong> &nbsp; | &nbsp;
              ↑ Upload: <strong>{{ .JSON.String "summary.totalUploadSpeed" }}</strong>
            </div>


        - type: custom-api
          title: "Downloading"
          cache: 30s
          url: n8n webhook
          template: |
            <h3 style="font-size: 1.5rem; margin-bottom: 1rem;">Download in corso ({{ .JSON.Int "summary.downloading" }})</h3>
            <ul style="list-style: none; padding: 0; display: flex; flex-direction: column; gap: 1rem;">
            {{ range .JSON.Array "downloading" }}
              <li style="background: #1e1e2f; padding: 1rem; border-radius: 0.75rem; box-shadow: 0 0 10px rgba(0,0,0,0.2);">
                <strong style="font-size: 1.1rem;">{{ .String "name" }}</strong>
                <div style="margin-top: 0.5rem; background: #333; border-radius: 10px; overflow: hidden;">
                  <div style="width: {{ .String "progress" }}; background: #4caf50; height: 10px;"></div>
                </div>
                <div style="margin-top: 0.5rem; font-size: 0.9rem; color: #ccc;">
                  Progress: <span style="color: #fff;">{{ .String "progress" }}</span><br/>
                  Velocità: <span style="color: #00c3ff;">{{ .String "downloadSpeed" }}</span>
                </div>
              </li>
            {{ else }}
              <li style="color: #ccc;">Nessun download in corso</li>
            {{ end }}
            </ul>

        - type: custom-api
          title: "Seed"
          cache: 5m
          url: n8n webhook
          template: |
            <h3 style="font-size: 1.5rem; margin-bottom: 1rem;">In Seed ({{ .JSON.Int "summary.seeding" }})</h3>
            <ul class="list list-gap-14 collapsible-container" data-collapse-after="1" style="padding: 0;">
            {{ range .JSON.Array "seeding" }}
              <li style="background: #1e1e2f; padding: 1rem; border-radius: 0.75rem; box-shadow: 0 0 10px rgba(0,0,0,0.2); list-style: none;">
                <strong style="font-size: 1.1rem;">{{ .String "name" }}</strong>
                <div style="margin-top: 0.5rem; font-size: 0.9rem; color: #ccc;">
                  Upload: <span style="color: #00ff94;">{{ .String "uploadSpeed" }}</span><br/>
                  Ratio: <span style="color: #ffd700;">{{ .String "ratio" }}</span>
                </div>
              </li>
            {{ else }}
              <li style="color: #ccc; list-style: none;">Nessun torrent in seed</li>
            {{ end }}
            </ul>


        - type: custom-api
          title: "Stopped"
          cache: 5m
          url: n8n webhook
          template: |
            <h3 style="font-size: 1.5rem; margin-bottom: 1rem;">Stoppati Recenti ({{ len (.JSON.Array "stopped") }})</h3>
            <ul class="list list-gap-14 collapsible-container" data-collapse-after="5" style="padding: 0;">
              {{ range .JSON.Array "stopped" }}
                <li style="background: #2a2a40; padding: 1rem; border-radius: 0.75rem;">
                  <strong style="font-size: 1.1rem;">{{ .String "name" }}</strong>
                  <div style="margin-top: 0.5rem; font-size: 0.9rem; color: #ccc;">
                    Stato: <span style="color: orange;">Stoppato</span><br/>
                  </div>
                </li>
              {{ else }}
                <li style="color: #ccc;">Nessun torrent stoppato</li>
              {{ end }}
            </ul>

        - type: custom-api
          title: "Errored"
          cache: 5m
          url: n8n webhook
          template: |
            <h3 style="font-size: 1.5rem; margin-bottom: 1rem;">Torrent in Errore ({{ len (.JSON.Array "errored") }})</h3>
            <ul class="list list-gap-14 collapsible-container" data-collapse-after="1" style="padding: 0;">
              {{ range .JSON.Array "errored" }}
                <li style="background: #3a1e1e; padding: 1rem; border-radius: 0.75rem;">
                  <strong style="font-size: 1.1rem;">{{ .String "name" }}</strong>
                  <div style="margin-top: 0.5rem; font-size: 0.9rem; color: #f88;">
                    Stato: <span style="color: red;">{{ .String "state" }}</span><br/>
                    Progresso: <span style="color: #ffcc00;">{{ .String "progress" }}</span>
                  </div>
                </li>
              {{ else }}
                <li style="color: #ccc;">Nessun torrent in errore</li>
              {{ end }}
            </ul>

  ```
</details>
Built by Sava, for the community.
