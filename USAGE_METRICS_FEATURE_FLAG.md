# Usage Metrics Feature Flag Guide

## Overview
The Usage Metrics feature can now be easily toggled on/off using an environment variable flag. This allows you to test the feature safely before enabling it in production.

---

## Feature Flag Configuration

### Environment Variable
```
ENABLE_USAGE_METRICS
```

### Default State
**Disabled** (`false`) - Usage Metrics endpoints will return 503 Service Unavailable

### Set to Enable
```bash
# In your .env file (development)
ENABLE_USAGE_METRICS=true

# Or set in your deployment/environment (production)
export ENABLE_USAGE_METRICS=true
```

---

## Affected Endpoints

When `ENABLE_USAGE_METRICS=false`, these endpoints will return **HTTP 503** with message:
> "Usage Metrics feature is currently disabled. Please contact your administrator."

1. **GET /management/stats/usage/inactive-users**
   - Shows users inactive for N days
   - Frontend: "Inactive Users" tab in Stats view

2. **GET /management/stats/usage/top-bottom-users**
   - Shows top and bottom users by report count
   - Frontend: "Stats" tab (disabled view)

3. **GET /management/stats/usage/coverage-metrics**
   - Shows overall usage percentage, trends, cluster coverage
   - Frontend: "Stats" tab (disabled view)

---

## Frontend Behavior

### When Feature is Disabled
- **Stats Tab** won't display metric cards
- **Load buttons** still visible (for testing)
- **Error message** shown: "Usage Metrics feature is currently disabled..."
- **All views** (Inactive, Top/Bottom, Coverage) handle the error gracefully

### When Feature is Enabled
- All three endpoints work normally
- Stats tab displays metrics and visualizations
- Coverage metrics, trends, and cluster data shown

---

## Deployment Steps

### Development/Testing
1. Update `.env` file:
   ```bash
   ENABLE_USAGE_METRICS=true
   ```

2. Restart backend service:
   ```bash
   # Kill existing process and restart
   python -m uvicorn app.main:app --reload
   ```

3. Test endpoints manually or through UI

### Production Rollout
1. Once testing is complete, update production environment:
   ```bash
   # AWS Lambda environment variable
   ENABLE_USAGE_METRICS=true
   
   # Or in Docker/ECS
   -e ENABLE_USAGE_METRICS=true
   ```

2. Restart the service

3. Monitor for errors in CloudWatch logs

4. Verify all three endpoints return proper responses

---

## Code Location

### Backend Flag Definition
**File:** `lami-report-backend/app/main.py` (lines ~413)
```python
# Feature Flags
# Set ENABLE_USAGE_METRICS=true to enable Usage Metrics endpoints
# Default: false (disabled for testing)
ENABLE_USAGE_METRICS = os.getenv('ENABLE_USAGE_METRICS', 'false').lower() == 'true'
```

### Endpoint Checks
**File:** `lami-report-backend/app/main.py`
- Line ~2817: `get_inactive_users()` endpoint check
- Line ~2923: `get_top_bottom_users()` endpoint check  
- Line ~3007: `get_coverage_metrics()` endpoint check

Each endpoint includes:
```python
if not ENABLE_USAGE_METRICS:
    raise HTTPException(
        status_code=503,
        detail="Usage Metrics feature is currently disabled. Please contact your administrator."
    )
```

---

## Testing Checklist

### Before Enabling in Production
- [ ] Test with flag `false` - verify 503 error message appears
- [ ] Test with flag `true` - verify all endpoints work
- [ ] Test error handling in frontend
- [ ] Verify cluster-specific access controls still work
- [ ] Check database query performance with full data
- [ ] Monitor CloudWatch logs for errors
- [ ] Test with different user roles (leader, admin, superuser)

### After Enabling in Production
- [ ] Monitor error rates in CloudWatch
- [ ] Check response times for `/management/stats/*` endpoints
- [ ] Review user feedback
- [ ] Track feature usage statistics
- [ ] Set up alerts if endpoints return errors

---

## Rollback Procedure

If issues occur after enabling:

1. **Quick Rollback:**
   ```bash
   # Set environment variable back to false
   export ENABLE_USAGE_METRICS=false
   
   # Restart service
   # Users will see "feature is currently disabled" message
   ```

2. **Verify Rollback:**
   - HTTP 503 responses confirm feature is disabled
   - Other management endpoints still functional

3. **Investigation:**
   - Check CloudWatch logs for specific errors
   - Review database query patterns
   - Check DynamoDB throttling metrics

---

## Future Enhancements

- [ ] Add timestamp tracking for when feature was last enabled/disabled
- [ ] Add logging of feature flag changes
- [ ] Create analytics dashboard for Usage Metrics usage patterns
- [ ] Add percentage rollout support (enable for X% of users)
- [ ] Create admin UI to toggle feature without code changes

---

## Quick Reference

| Scenario | Action | Result |
|----------|--------|--------|
| **Development Testing** | `ENABLE_USAGE_METRICS=true` in `.env` | Full access to all endpoints |
| **Production (Pre-Release)** | `ENABLE_USAGE_METRICS=false` | 503 error, feature disabled |
| **Production (Released)** | `ENABLE_USAGE_METRICS=true` | Full access to all endpoints |
| **Rollback Needed** | `ENABLE_USAGE_METRICS=false` | Quick disable without code changes |
