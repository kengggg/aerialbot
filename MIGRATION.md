# Migration Plan: Merge v2api Branch to Master

## Background

The `v2api` branch has been running in production successfully for 2+ years and contains significant improvements over the current `master` branch. With the original maintainer appearing inactive, it's time to merge the stable `v2api` branch to `master` to establish it as the main development branch.

## Current State Analysis

### Branch Comparison
- **master**: Mastodon-only bot (Twitter support removed in fba8731)
- **v2api**: 5 commits ahead, dual Twitter/Mastodon support with modern APIs
- **Common ancestor**: 4b7107da8005622f181264c9ecf648cd70f0e51a

### Key Improvements in v2api
- ✅ Twitter API v2 support with tweepy 4.14.0
- ✅ Reverse geocoding services (Google Maps + Nominatim)
- ✅ Enhanced location services integration
- ✅ Simplified configuration management
- ✅ Dual Twitter/Mastodon posting capability
- ✅ 2+ years of proven production stability

### Files Modified
```
.gitignore             |   4 +-
README.md              |  25 ++++---
aerialbot.py           | 174 +++++++++++++++++++++++++++++--------------------
aerialbot_utilities.py |  45 +++++++++++++
config.sample.ini      |  80 +++++++++++++++++------
requirements.txt       |  30 +++++----
shapefiles/README.md   |   2 +-
```

## Migration Plan

### Phase 1: Pre-Migration Preparation
- [ ] Create safety backup of current master branch
- [ ] Verify repository state and dependencies
- [ ] Test dependency installation in clean environment
- [ ] Document current configuration requirements

### Phase 2: Merge Strategy
**Recommended Approach: Force Merge**
- Rationale: v2api represents the actual production state
- Benefits: Clean transition, reflects real development history
- Risk mitigation: Backup branch created for rollback

**Alternative approaches considered:**
- Standard merge commit (may create conflicts)
- Rebase and merge (more complex, potential conflicts)

### Phase 3: Merge Execution
```bash
# Safety backup
git branch master-backup-$(date +%Y%m%d)

# Force merge v2api to master
git checkout master
git reset --hard v2api
git push origin master --force-with-lease
```

### Phase 4: Post-Merge Validation
- [ ] Dependency installation verification
- [ ] Configuration structure validation
- [ ] Basic functionality testing
- [ ] Code quality checks (syntax, imports)
- [ ] API connectivity tests (if credentials available)

### Phase 5: Repository Maintenance
- [ ] Update documentation if needed
- [ ] Clean up merged branches
- [ ] Tag release as v2.0.0
- [ ] Update repository metadata

## Risk Assessment

### High Priority Risks
1. **Dependency Conflicts** - v2api requires newer packages
   - *Mitigation*: Test in clean environment first
   - *Fallback*: Restore from backup branch

2. **API Configuration Changes** - New Twitter API requirements
   - *Mitigation*: Document credential requirements clearly
   - *Fallback*: Mastodon-only mode remains functional

### Medium Priority Risks
1. **Configuration Breaking Changes** - Simplified config format
   - *Mitigation*: Provide migration guide
   - *Fallback*: Clear sample configuration provided

2. **Feature Regressions** - Some master features might be removed
   - *Mitigation*: Feature comparison audit
   - *Fallback*: Cherry-pick critical missing features

## Success Criteria

### Must Have ✅
- [ ] All Python files compile without errors
- [ ] Dependencies install from requirements.txt
- [ ] Basic image generation functionality works
- [ ] Mastodon posting capability maintained

### Should Have 🎯
- [ ] Twitter posting works with valid credentials
- [ ] Location services function correctly
- [ ] Configuration validation working
- [ ] Logging system operational

### Nice to Have 🌟
- [ ] All CLI options functional
- [ ] Documentation up-to-date
- [ ] Repository clean and organized

## Rollback Plan

If critical issues arise:
1. Checkout backup: `git checkout master-backup-YYYYMMDD`
2. Reset master: `git branch -D master && git checkout -b master`
3. Force push: `git push origin master --force-with-lease`
4. Investigate issues in separate branch

## Dependencies Added/Updated

```diff
+ tweepy==4.14.0
+ geopy==2.3.0
  blurhash==1.1.4
  certifi==2022.12.7
  charset-normalizer==2.1.1
  configobj==5.0.6
  # ... (other dependencies updated for security/compatibility)
```

## Detailed Technical Changes

### Twitter API v2 Migration
The v2api branch implements modern Twitter API v2 support:

```python
# New Tweeter class with dual API support
class Tweeter:
    def __init__(self, consumer_key, consumer_secret, access_token, access_token_secret):
        auth = tweepy.OAuth1UserHandler(consumer_key, consumer_secret)
        auth.set_access_token(access_token, access_token_secret)
        # API v1.1 for media upload
        self.api = tweepy.API(auth)
        # API v2 for tweet creation
        self.client = tweepy.Client(...)
```

### Location Services
New reverse geocoding capabilities:

```python
# Google Maps integration
def get_location_googlemaps(geopoint, language, api_key):
    url = f"https://maps.googleapis.com/maps/api/geocode/json?latlng={geopoint.lat},{geopoint.lon}..."
    # Returns (full_name, country)

# Nominatim (OSM) integration  
def get_location_nominatim(geopoint):
    geolocator = Nominatim(user_agent="aerialbot")
    location = geolocator.reverse((geopoint.lat, geopoint.lon), language='th', zoom=14)
    # Returns (full_name, country)
```

### Configuration Changes
Simplified configuration structure removes complex multi-config merging:

```diff
- # Complex multi-config support removed
- config_paths = args.config_paths
- for config_path in args.config_paths:
-     loaded_config = ConfigObj(config_path, unrepr=True)
-     config.merge(loaded_config)

+ # Single config file approach
+ config = ConfigObj(args.config_path, unrepr=True)
```

### New Configuration Sections
```ini
[LOCATIONSERVICES]
reverse_geocoding_service = "googlemaps"  # or "nominatim"
google_maps_api_key = "YOUR_API_KEY"
google_maps_reverse_geocoding_language = "en"

[TWITTER]
consumer_key = "YOUR_CONSUMER_KEY"
consumer_secret = "YOUR_CONSUMER_SECRET"
access_token = "YOUR_ACCESS_TOKEN"
access_token_secret = "YOUR_ACCESS_TOKEN_SECRET"
tweet_text = "Your tweet template with {location_full_name}"
include_location_in_metadata = True
```

## Implementation Checklist

### Pre-Migration
- [ ] Review and approve migration plan
- [ ] Schedule maintenance window
- [ ] Prepare rollback procedures
- [ ] Test environment setup

### Migration Day
- [ ] Execute backup creation
- [ ] Perform merge operation
- [ ] Run validation tests
- [ ] Update documentation
- [ ] Tag release

### Post-Migration
- [ ] Monitor for issues
- [ ] Update CI/CD if applicable
- [ ] Communicate changes to users
- [ ] Archive old branches

## Timeline Estimate
- **Preparation**: 30 minutes
- **Execution**: 15 minutes  
- **Validation**: 45 minutes
- **Cleanup**: 15 minutes
- **Total**: ~2 hours

## Validation Commands

### Dependency Check
```bash
python -m venv test-env
source test-env/bin/activate
pip install -r requirements.txt
```

### Syntax Validation
```bash
python -m py_compile aerialbot.py
python -m py_compile aerialbot_utilities.py
```

### Basic Functionality Test
```bash
cp config.sample.ini test-config.ini
# Edit test-config.ini with test values
python aerialbot.py test-config.ini --help
python aerialbot.py test-config.ini --point="37.7749,-122.4194"
```

## Notes
- This migration has been delayed too long - v2api has proven stability
- Original maintainer appears inactive since March 2025
- Current production instances already running v2api successfully
- This migration formalizes the existing production reality

---

**Priority**: High  
**Complexity**: Medium  
**Impact**: High - Establishes active maintenance and modern API support

## Status: Planning Phase
This document serves as the comprehensive planning documentation for the v2api to master migration. All implementation steps are documented but not yet executed.