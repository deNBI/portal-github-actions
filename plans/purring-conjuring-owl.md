# Fix "Unrecognized named-value: 'secrets'" in reusable workflow call

## Context
The user is encountering a GitHub Actions error in a caller workflow (`simplevm-draft-repo/.github/workflows/release_image.yml`) when calling the reusable workflow `denbi/portal-github-actions/.github/workflows/release-image.yml`.

The error `Unrecognized named-value: 'secrets'` occurs because the caller is attempting to use the `secrets` context within the `with` block of a reusable workflow call:

```yaml
with:
  matrix_room: ${{ secrets.MATRIX_CRITICAL_ROOM }}
  matrix_token: ${{ secrets.matrix_access_token }}
```

In GitHub Actions, the `secrets` context is not available in the `with` block of a reusable workflow call. Secrets must be passed using the `secrets` keyword.

## Implementation Plan

### 1. Update Reusable Workflow (`portal-github-actions/.github/workflows/release-image.yml`)
The reusable workflow currently expects `matrix_room` and `matrix_token` as inputs. These should be moved to the `secrets` section to allow them to be passed securely and to resolve the caller's error.

- **Remove from `inputs`**:
  - `matrix_room`
  - `matrix_token`
- **Add to `secrets`**:
  - `MATRIX_ROOM` (optional/required: false)
  - `MATRIX_ACCESS_TOKEN` (already exists, keep it)
- **Update `Scan Image` step**:
  - Change `matrix_room: ${{ inputs.matrix_room }}` to `matrix_room: ${{ secrets.MATRIX_ROOM }}`
  - Change `matrix_token: ${{ secrets.MATRIX_ACCESS_TOKEN }}` (already correct, but ensure it uses the secret)

### 2. Recommend Changes to Caller Workflow (`simplevm-draft-repo/.github/workflows/release_image.yml`)
Although the primary fix is in the reusable workflow, the caller must also be updated to move the secrets from the `with` block to the `secrets` block.

- **Remove from `with`**:
  - `matrix_room: ${{ secrets.MATRIX_CRITICAL_ROOM }}`
  - `matrix_token: ${{ secrets.matrix_access_token }}`
- **Add to `secrets`**:
  - `MATRIX_ROOM: ${{ secrets.MATRIX_CRITICAL_ROOM }}`
  - `MATRIX_ACCESS_TOKEN: ${{ secrets.matrix_access_token }}` (already exists)

## Verification Plan
1. Verify that the syntax in `portal-github-actions/.github/workflows/release-image.yml` is correct after modifications.
2. (If possible) Trigger the caller workflow in `simplevm-draft-repo` to ensure the `secrets` error is gone and that Matrix notifications are still working.
