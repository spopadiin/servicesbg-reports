# ServicesBG Migration Media Policy Report

Updated: 2026-06-29T10:49:04+03:00

## Summary
The missing media inspector found 40 missing media references. All 40 are YouTube/video references stored in `fl_listing_photos`, not missing image files.

Policy outcome:
- Missing image files: 0.
- Preserve video reference without attachment: 40.
- Promote first existing image: 0.
- Assign generic placeholder: 0.
- Skip secondary missing image: 0.
- Critical missing media directory: 0.

Placeholder strategy:
- Default reference: `servicesbg_generic_placeholder_reference`.
- Final design asset is not required yet.
- Category-specific placeholders can be introduced later.

## Policy

| Case | Action |
| --- | --- |
| Secondary image missing | Skip and log. |
| Primary image missing and other images exist | Promote first existing local image. |
| Only image missing | Assign ServicesBG placeholder. |
| Listing has no media | No action. |
| Video reference | Preserve video metadata; do not import as attachment. |
| Missing media directory | Critical blocker. |

## Missing References

| photo_id | listing_id | provider_id | category_id | filename | expected path/ref | issue | primary | secondary | only image | other images | other count |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- | --- | --- | --- | ---: |
| 4156 | 15 | 19 | 2073 | 8aHBMRiZI-o | youtube:8aHBMRiZI-o | video_reference_non_file | no | no | no | yes | 2 |
| 4187 | 15 | 19 | 2073 | 8aHBMRiZI-o | youtube:8aHBMRiZI-o | video_reference_non_file | no | no | no | yes | 2 |
| 4157 | 25 | 16 | 2127 | 9gImm27YW_g | youtube:9gImm27YW_g | video_reference_non_file | no | no | no | yes | 10 |
| 4188 | 25 | 16 | 2127 | 9gImm27YW_g | youtube:9gImm27YW_g | video_reference_non_file | no | no | no | yes | 10 |
| 4158 | 26 | 16 | 2131 | ZOczr9WCVig | youtube:ZOczr9WCVig | video_reference_non_file | no | no | no | yes | 15 |
| 4189 | 26 | 16 | 2131 | ZOczr9WCVig | youtube:ZOczr9WCVig | video_reference_non_file | no | no | no | yes | 15 |
| 4159 | 27 | 16 | 2266 | iFEi50yaqRo | youtube:iFEi50yaqRo | video_reference_non_file | no | no | no | yes | 12 |
| 4190 | 27 | 16 | 2266 | iFEi50yaqRo | youtube:iFEi50yaqRo | video_reference_non_file | no | no | no | yes | 12 |
| 4160 | 28 | 16 | 2106 | wzSoKHt9YF8 | youtube:wzSoKHt9YF8 | video_reference_non_file | no | no | no | yes | 13 |
| 4191 | 28 | 16 | 2106 | wzSoKHt9YF8 | youtube:wzSoKHt9YF8 | video_reference_non_file | no | no | no | yes | 13 |
| 4161 | 273 | 19 | 2283 | PPckOumx7h4 | youtube:PPckOumx7h4 | video_reference_non_file | no | no | no | yes | 5 |
| 4192 | 273 | 19 | 2283 | PPckOumx7h4 | youtube:PPckOumx7h4 | video_reference_non_file | no | no | no | yes | 5 |
| 4162 | 283 | 19 | 2187 | iDHtxroc_CU | youtube:iDHtxroc_CU | video_reference_non_file | no | no | no | yes | 1 |
| 4193 | 283 | 19 | 2187 | iDHtxroc_CU | youtube:iDHtxroc_CU | video_reference_non_file | no | no | no | yes | 1 |
| 4163 | 286 | 19 | 2200 | tB9bjPwmieA | youtube:tB9bjPwmieA | video_reference_non_file | no | no | no | yes | 1 |
| 4194 | 286 | 19 | 2200 | tB9bjPwmieA | youtube:tB9bjPwmieA | video_reference_non_file | no | no | no | yes | 1 |
| 4164 | 341 | 19 | 2247 | algqpxIbhC8 | youtube:algqpxIbhC8 | video_reference_non_file | no | no | no | no | 0 |
| 4195 | 341 | 19 | 2247 | algqpxIbhC8 | youtube:algqpxIbhC8 | video_reference_non_file | no | no | no | no | 0 |
| 4165 | 354 | 19 | 2217 | rWE5kYuoYE0 | youtube:rWE5kYuoYE0 | video_reference_non_file | no | no | no | yes | 1 |
| 4196 | 354 | 19 | 2217 | rWE5kYuoYE0 | youtube:rWE5kYuoYE0 | video_reference_non_file | no | no | no | yes | 1 |
| 4166 | 359 | 19 | 2212 | 05we2g3Edgs | youtube:05we2g3Edgs | video_reference_non_file | no | no | no | yes | 1 |
| 4197 | 359 | 19 | 2212 | 05we2g3Edgs | youtube:05we2g3Edgs | video_reference_non_file | no | no | no | yes | 1 |
| 4167 | 404 | 19 | 2246 | UigOPfLexJ0 | youtube:UigOPfLexJ0 | video_reference_non_file | no | no | no | yes | 1 |
| 4198 | 404 | 19 | 2246 | UigOPfLexJ0 | youtube:UigOPfLexJ0 | video_reference_non_file | no | no | no | yes | 1 |
| 4168 | 736 | 19 | 2153 | CV0NUYzB5lA | youtube:CV0NUYzB5lA | video_reference_non_file | no | no | no | yes | 1 |
| 4199 | 736 | 19 | 2153 | CV0NUYzB5lA | youtube:CV0NUYzB5lA | video_reference_non_file | no | no | no | yes | 1 |
| 4169 | 748 | 19 | 2153 | Ae0csCp5KXU | youtube:Ae0csCp5KXU | video_reference_non_file | no | no | no | yes | 1 |
| 4200 | 748 | 19 | 2153 | Ae0csCp5KXU | youtube:Ae0csCp5KXU | video_reference_non_file | no | no | no | yes | 1 |
| 4170 | 1765 | 19 | 2172 | xD5UpyvMcGw | youtube:xD5UpyvMcGw | video_reference_non_file | no | no | no | yes | 4 |
| 4201 | 1765 | 19 | 2172 | xD5UpyvMcGw | youtube:xD5UpyvMcGw | video_reference_non_file | no | no | no | yes | 4 |
| 4171 | 1771 | 19 | 2188 | 8BGJp8tiiYI | youtube:8BGJp8tiiYI | video_reference_non_file | no | no | no | yes | 4 |
| 4202 | 1771 | 19 | 2188 | 8BGJp8tiiYI | youtube:8BGJp8tiiYI | video_reference_non_file | no | no | no | yes | 4 |
| 4172 | 1776 | 19 | 2186 | asx_217RCd8 | youtube:asx_217RCd8 | video_reference_non_file | no | no | no | yes | 6 |
| 4203 | 1776 | 19 | 2186 | asx_217RCd8 | youtube:asx_217RCd8 | video_reference_non_file | no | no | no | yes | 6 |
| 4173 | 1778 | 19 | 2188 | ewdzUOu8UM0 | youtube:ewdzUOu8UM0 | video_reference_non_file | no | no | no | yes | 2 |
| 4204 | 1778 | 19 | 2188 | ewdzUOu8UM0 | youtube:ewdzUOu8UM0 | video_reference_non_file | no | no | no | yes | 2 |
| 4174 | 1779 | 19 | 2186 | eUtJrbLOEbs | youtube:eUtJrbLOEbs | video_reference_non_file | no | no | no | yes | 3 |
| 4205 | 1779 | 19 | 2186 | eUtJrbLOEbs | youtube:eUtJrbLOEbs | video_reference_non_file | no | no | no | yes | 3 |
| 4175 | 1783 | 35 | 2089 | Ho5wu4Rc9Xo | youtube:Ho5wu4Rc9Xo | video_reference_non_file | no | no | no | yes | 6 |
| 4206 | 1783 | 35 | 2089 | Ho5wu4Rc9Xo | youtube:Ho5wu4Rc9Xo | video_reference_non_file | no | no | no | yes | 6 |

## Blockers
None.

## Warnings
- `video_references_are_not_attachment_files`
