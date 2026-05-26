# Corporate Alert Banner (SPFx 1.20)

Tenant-wide corporate alert banner. Two SPFx components ship in a single solution:

| Component | Type | Purpose |
| --- | --- | --- |
| `CorporateAlertBannerApplicationCustomizer` | Application Customizer (extension) | Renders the banner in the `Top` placeholder on every modern page. Auto-deployed tenant-wide. |
| `AlertBannerConfigWebPart` | Web Part | Lives on the home page of the source site. Property pane edits a single item in the central list and publishes it. |

Both components share the **`CorporateAlertBanner`** SharePoint list on the source site (root site by default). The extension reads from it; the web part writes to it.

---

<img width="1898" height="445" alt="image" src="https://github.com/user-attachments/assets/02429672-00da-4194-b822-850c674d07fa" />

<img width="1893" height="380" alt="image" src="https://github.com/user-attachments/assets/0a8205d3-437e-478a-8d85-b7fc10a92542" />

<img width="1910" height="538" alt="image" src="https://github.com/user-attachments/assets/ad30f43d-cfd0-4497-9ffb-e10da6004981" />



## 1. One-time setup on the source site

Before the solution is useful, create the central list on the **root site** (`/`) of the tenant.

**List name:** `CorporateAlertBanner` (must match exactly — title-based REST lookup)

**Columns:**

| Internal name | Display name | Type | Notes |
| --- | --- | --- | --- |
| `Title` | Title | Single line of text | Built-in; not used in UI |
| `Enabled` | Enabled | Yes/No | Required |
| `Icon` | Icon | Choice | Choices: `Information`, `Warning`, `Maintenance`, `Error`, `Success` |
| `Message` | Message | Multiple lines of text (plain) | Required |
| `BarColor` | Bar color | Single line of text | Hex, e.g. `#0078d4` |
| `FontColor` | Font color | Single line of text | Hex, e.g. `#ffffff` |
| `FontFamily` | Font family | Single line of text | e.g. `Segoe UI` |
| `FontSize` | Font size | Number | Integer px, 10–28 |
| `UrlLink` | URL link | Single line of text | Absolute URL or blank |

Permissions: grant **Contribute** to anyone allowed to edit the banner; **Read** to Everyone except external users.

Tip — provision with PnP PowerShell:

```powershell
Connect-PnPOnline -Url https://<tenant>.sharepoint.com -Interactive
New-PnPList -Title "CorporateAlertBanner" -Template GenericList
Add-PnPField -List "CorporateAlertBanner" -DisplayName "Enabled"     -InternalName "Enabled"     -Type Boolean
Add-PnPField -List "CorporateAlertBanner" -DisplayName "Icon"        -InternalName "Icon"        -Type Choice -Choices "Information","Warning","Maintenance","Error","Success"
Add-PnPField -List "CorporateAlertBanner" -DisplayName "Message"     -InternalName "Message"     -Type Note
Add-PnPField -List "CorporateAlertBanner" -DisplayName "BarColor"    -InternalName "BarColor"    -Type Text
Add-PnPField -List "CorporateAlertBanner" -DisplayName "FontColor"   -InternalName "FontColor"   -Type Text
Add-PnPField -List "CorporateAlertBanner" -DisplayName "FontFamily"  -InternalName "FontFamily"  -Type Text
Add-PnPField -List "CorporateAlertBanner" -DisplayName "FontSize"    -InternalName "FontSize"    -Type Number
Add-PnPField -List "CorporateAlertBanner" -DisplayName "UrlLink"     -InternalName "UrlLink"     -Type Text
```

---

## 2. Build the .sppkg

```powershell
npm install
gulp bundle --ship
gulp package-solution --ship
```

Output: `sharepoint/solution/corporate-alert-banner.sppkg`.

> **Node version note:** SPFx 1.20 targets Node 18 / 22 LTS. If you are on Node 24, switch with `nvm use 22` before installing.

---

## 3. Deploy

1. Upload `corporate-alert-banner.sppkg` to your **tenant App Catalog** (`/sites/appcatalog`).
2. When prompted, check **"Make this solution available to all sites in the organization"** and click **Deploy**. The Application Customizer is registered tenant-wide automatically via `clientsideinstance.xml`.
3. On the **root site** (`/`), add the **Corporate Alert Banner Config** web part to the home page.
4. Open its property pane, configure the banner, toggle **Enabled**, then click **Save and publish**.
5. Hard-refresh any other site — the banner appears at the top.

---

## 4. Local debug (workbench)

`config/serve.json` is pre-wired for the Application Customizer. Edit the `pageUrl` to point at a real page in your tenant, then run:

```powershell
gulp serve --nobrowser
```

Open the page with `?debug=true&noredir=true&debugManifestsFile=https://localhost:4321/temp/manifests.js` appended (gulp prints the full URL).

---

## 5. Files

```
src/extensions/corporateAlertBanner/
  CorporateAlertBannerApplicationCustomizer.ts   # entrypoint, reads list and renders Top
  AlertConfigService.ts                          # shared REST CRUD against the list
  IAlertBannerConfig.ts                          # shared types + defaults + icon map
  components/AlertBanner.tsx                     # banner render
sharepoint/assets/
  elements.xml                                   # CustomAction for tenant registration
  clientsideinstance.xml                         # Tenant-wide auto-instance
src/webparts/alertBannerConfig/
  AlertBannerConfigWebPart.ts                    # property pane + Save handler
  components/AlertBannerConfig.tsx               # preview UI
```

The web part imports `AlertConfigService` and the shared `AlertBanner` component from the extension folder — one source of truth for both write and render.
