# Work Report Usage Statistics Dashboard - Quick Start Guide

## What's New?

The Management Modal now includes a comprehensive **Usage Metrics Dashboard** that helps cluster heads and super users monitor:
1. **Inactive Users** - Who hasn't submitted reports recently
2. **Top/Bottom Performers** - Users ranked by report volume  
3. **Coverage Metrics** - Overall app adoption and usage trends

## How to Access

1. Open the **Management Modal** (in the app navigation)
2. Click on the **Stats** tab
3. You'll see two sub-tabs:
   - **Work Statistics** (original heatmap/KPI reports)
   - **Usage Metrics** (NEW - the new dashboard)
4. Click on **Usage Metrics** to view the three data views

## The Three Views

### 1. Inactive Users
**What it shows**: Users who haven't created reports in the last 30 days (configurable)

**How to use it**:
- Set the threshold (default: 30 days) using the number input
- Click "Load Data" to fetch the latest information
- Review the table to see:
  - User names and emails
  - Which cluster they belong to
  - When they last submitted a report
  - How many days they've been inactive

**Why it matters**: 
- Identify users who need support or reminders
- Track engagement across your clusters
- Reach out to inactive team members

**Action items**:
- Send personalized follow-ups to inactive users
- Schedule support sessions
- Provide additional training if needed

---

### 2. Top/Bottom Users
**What it shows**: The 10 most active and 10 least active users (configurable)

**How to use it**:
- Adjust the limit (default: 10) to see more/fewer users
- Click "Refresh" to get the latest data
- Two columns show:
  - **Left side**: Top performers (ranked #1, #2, #3...)
  - **Right side**: Bottom performers (ranked from bottom)
  - **Report Count**: Total number of work reports each user created

**Why it matters**:
- Recognize your top contributors
- Understand adoption variation across the team
- Identify users who may need additional support

**Action items**:
- Acknowledge top performers and their contributions
- Create peer mentoring programs (top performance lessons)
- Provide targeted support to users with fewer reports

---

### 3. Coverage Metrics
**What it shows**: Overall app usage statistics and trends

**Summary Cards** (Top section):
- **Overall Usage Percentage**: % of users who have submitted at least one report
  - Color-coded: Green (70%+), Orange (50-70%), Red (<50%)
- **Total Reports**: Cumulative count of all work reports ever created
- **Coverage Trend**: Direction indicator showing if usage is improving (↑), declining (↓), or stable (→)
- **Active Users**: Count of users with at least one report

**Coverage by Cluster** (Table):
- See usage percentage for each department/cluster
- View total vs. active users per cluster
- Visual progress bars show cluster-level adoption

**Monthly Trend Chart** (Bottom):
- Bar chart showing reports submitted per month
- Last 12 months of data
- Helps identify seasonal patterns or adoption milestones

**Why it matters**:
- Measure overall team adoption of the app
- Track improvement over time
- Identify struggling clusters vs. high performers
- Make data-driven decisions about training

**Action items**:
- Set adoption targets based on industry benchmarks
- Create cluster-specific improvement plans
- Celebrate milestone achievements
- Launch recruitment campaigns for adoption

---

## Common Scenarios

### Scenario 1: "Why is adoption slow?"
1. Go to **Coverage Metrics** view
2. Check **Overall Usage Percentage** and **Monthly Trend**
3. Look at **Coverage by Cluster** to find underperforming teams
4. Switch to **Inactive Users** to identify specific people to support
5. Take action: reach out, provide training, remove barriers

### Scenario 2: "Who are our best performers?"
1. Go to **Top/Bottom Users** view
2. Increase the limit to see top 20-30 users
3. Review the list
4. Consider them for case studies, presentations, or peer mentoring

### Scenario 3: "Is adoption improving?"
1. Go to **Coverage Metrics** view
2. Check the **Coverage Trend** indicator
3. Look at the **Monthly Trend** chart
4. Compare current Overall Usage % to previous month
5. If improving (↑): continue current initiatives, celebrate wins
6. If declining (↓): investigate causes, adjust strategy
7. If stable (→): maintain current approach or introduce new initiatives

### Scenario 4: "One cluster is lagging"
1. Go to **Coverage Metrics** → **Coverage by Cluster**
2. Find the cluster with lowest usage %
3. Switch to **Inactive Users** and filter by that cluster
4. Identify the specific users not participating
5. Create an action plan specific to that cluster's needs

---

## Tips & Tricks

### Data Refresh
- Each view has a "Load Data" or "Refresh" button
- Click to get the latest information
- Data is calculated in real-time from your database

### Flexible Thresholds
- **Inactive Users**: Change the day threshold (try 7, 14, 30, 60, 90)
- **Top/Bottom Users**: Adjust limit to see more users (up to 50)
- Experiment to find insights relevant to your goals

### Visual Cues
- **Red text/numbers**: Indicate problems or areas needing attention
- **Green**: Positive metrics or good performance
- **Blue/Orange**: Ranking indicators in Top/Bottom Users
- **Progress bars color**: Green (good), Orange (warning), Red (critical)

### Export Data
- You can take screenshots of tables for reports
- Or use browser dev tools to copy table data
- (Future: CSV export feature planned)

---

## Interpreting the Data

### Overall Usage Percentage
- **80%+**: Excellent adoption, keep it up!
- **60-80%**: Good adoption, room for improvement
- **40-60%**: Significant opportunity for growth
- **<40%**: Urgent need for adoption initiatives

### Coverage Trend
- **↑ Improved**: More users submitting reports month-over-month
- **→ Neutral**: Stable usage, no significant change
- **↓ Decreased**: Usage declining, investigate cause

### Days Inactive (in Inactive Users view)
- **<30 days**: Recently active or just taking a break
- **30-60 days**: Should be contacted soon
- **60-90 days**: Needs immediate outreach and support
- **>90 days**: Consider whether role still requires app usage

---

## Permissions

You can access this dashboard if you have:
- **Superuser** role (access to all clusters), OR
- **Cluster Leader** role (access to your cluster's data)

You'll only see data for clusters you're authorized to manage.

---

## Frequently Asked Questions

**Q: Why do I see different numbers than expected?**
A: The dashboard queries your live DynamoDB data. Changes should appear immediately after clicking "Load Data" or "Refresh."

**Q: Can I filter by cluster?**
A: The dashboard respects your cluster access level automatically. Superusers see all clusters; leaders see only their own.

**Q: How often should I check these metrics?**
A: Monthly reviews are recommended to track trends and plan initiatives. Weekly checks can help identify urgent issues.

**Q: What if all users are showing as inactive?**
A: Increase the day threshold to 180+ to see if there was recent activity. Or check if there's a data sync issue.

**Q: Can I share these reports?**
A: Take screenshots or use browser tools to copy data. Official export feature coming soon!

**Q: What actions can I take with this data?**
A: The data helps you identify who needs support, recognize performers, set targets, and measure progress on adoption goals.

---

## Best Practices

1. **Review Monthly**: Check at least once per month to track trends
2. **Set Goals**: Use baseline numbers to set realistic adoption targets
3. **Segment By Cluster**: Understand variations across different teams
4. **Identify Champions**: Find top performers to help with peer mentoring
5. **Take Action**: Use insights to inform training, support, and communications
6. **Track Results**: Monitor whether your efforts are improving the metrics
7. **Share Results**: Communicate progress to your team (celebrate wins!)
8. **Adjust Strategy**: If trendsis negative, investigate and adjust approach

---

## Contact & Support

For questions about:
- **Interpretation**: Review this guide or consult your manager
- **Technical Issues**: Check dashboard error messages or contact IT
- **Feature Requests**: Those are very welcome! Document what would help

---

## Summary Dashboard Checklist

When accessing the dashboard, ask yourself:

- [ ] What is our current Overall Usage Percentage?
- [ ] Is coverage trending up, down, or flat?
- [ ] Which clusters have the highest/lowest usage?
- [ ] Who are the top 5 performers in our organization?
- [ ] How many users are inactive for 30+ days?
- [ ] What is the monthly trend - are we improving?
- [ ] Which specific users need follow-up?
- [ ] What concrete actions will I take based on this data?

Good luck with your app adoption initiatives! The data is your tool to success.
