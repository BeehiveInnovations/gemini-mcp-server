# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

**ZENODE MCP SERVER**: This is the Node.js/TypeScript port of the original Python Zen MCP Server. The Python code in this repository is maintained for reference and feature parity purposes only.

**ACTIVE DEVELOPMENT**: Focus on the `zenode/` directory which contains the Node.js implementation. The Python code serves as reference for understanding the original architecture and ensuring feature parity.

**MIGRATION GOAL**: Convert this entire Python-based Zen MCP Server to Node.js while maintaining all functionality and architecture patterns.

**DEVELOPER CONTEXT**: The developer working on this project is a Python novice with strong JavaScript/TypeScript/Node.js/bash experience. When working with the existing Python code, explain Python concepts, ecosystem, toolchain, and syntax as if teaching someone who understands JS/TS/Node.js but is new to Python.

This repository contains both:
- **zenode/** - The active Node.js/TypeScript implementation 
- **Python files** - Reference implementation for feature parity (read-only for comparison)

The goal is to maintain the Node.js implementation while preserving:
- All AI tools (chat, thinkdeep, codereview, precommit, debug, analyze, testgen, etc.)
- Multi-provider architecture (Gemini, OpenAI, OpenRouter, custom endpoints)
- Conversation threading with Redis
- Docker-based deployment
- MCP protocol compliance

## Zenode MCP Server File Access & Workspace Guide

### 🚨 CRITICAL: How to Access Files in Zenode Tools

**The zenode MCP server runs in Docker containers and CANNOT access files directly from your local filesystem.** Here's how to properly provide files to zenode tools:

#### Container File Path Mapping
```bash
# Your local file system:
/Users/edunc/Documents/project/image.jpg

# Inside zenode container:
/workspace/Documents/project/image.jpg
```

#### ✅ CORRECT Way to Use Zenode Tools with Files

**For Images:**
```bash
# ❌ WRONG - zenode cannot access this:
zenode:chat "Analyze this image" --files ["/Users/edunc/Desktop/screenshot.png"]

# ✅ CORRECT - use container path:
zenode:chat "Analyze this image" --files ["/workspace/Desktop/screenshot.png"]
```

**For Code Files:**
```bash
# ❌ WRONG:
zenode:analyze --files ["/Users/edunc/Documents/myproject/src/app.js"]

# ✅ CORRECT:
zenode:analyze --files ["/workspace/Documents/myproject/src/app.js"]
```

#### Docker Volume Mounts (zenode/docker-compose.yml)
```yaml
volumes:
  - ${HOME}:/workspace:rw  # Your home directory → /workspace in container
  - ${MCP_WORKSPACE:-./workspace}:/workspace:ro  # Alternative workspace mount
```

#### Quick File Path Conversion
```bash
# Replace your home directory path with /workspace
/Users/edunc/anything/file.ext  →  /workspace/anything/file.ext
/home/username/anything/file.ext  →  /workspace/anything/file.ext
```

#### Verify File Access
```bash
# Always verify files are accessible before analysis:
zenode:gopher --action file_exists --path "/workspace/path/to/your/file.jpg"
```

### 🎯 Image Analysis with Zenode

#### Complete Workflow for Image Analysis
```bash
# 1. Copy image to accessible location (optional)
cp /path/to/image.jpg ~/Desktop/  

# 2. Use zenode with container path
zenode:chat "Analyze this face image in detail" --files ["/workspace/Desktop/image.jpg"]

# 3. For comprehensive analysis
zenode:analyze --files ["/workspace/Desktop/image.jpg"] --prompt "Detailed image analysis"
```

#### Automatic Vision Model Selection
- When you provide images to zenode tools, they automatically select vision-capable models
- Default vision model: `openai/gpt-4o` (configurable via `DEFAULT_VISION_MODEL`)
- Supports: OpenRouter, OpenAI, Gemini vision models

### 🔧 Zenode Setup & Configuration

#### Required Environment Variables
```bash
# At minimum, set one API key:
export OPENROUTER_API_KEY="your_key_here"  # Recommended for vision
export OPENAI_API_KEY="your_key_here"      # For direct OpenAI access
export GEMINI_API_KEY="your_key_here"      # For Google models

# Optional vision model override:
export DEFAULT_VISION_MODEL="openai/gpt-4o"  # Default is already gpt-4o
```

#### Start Zenode Server
```bash
cd zenode/
./run-server.sh  # Starts Docker containers with proper volume mounts
```

#### Verify Setup
```bash
# Check if zenode can access your files:
zenode:gopher --action list_directory --path "/workspace"

# Test image analysis:
zenode:chat "What do you see?" --files ["/workspace/Desktop/test-image.jpg"]
```

### 📋 Zenode Tools Reference

#### Image-Capable Tools
- **zenode:seer** - 🔮 DEDICATED VISION TOOL for all image analysis tasks
- **zenode:chat** - General conversation (delegates images to seer)
- **zenode:analyze** - Code/file analysis (delegates images to seer)
- **zenode:gopher** - File system access and verification

#### Non-Image Tools
- **zenode:thinkdeep** - Extended reasoning (text only)
- **zenode:codereview** - Code review (code files only)
- **zenode:debug** - Debugging assistance
- **zenode:testgen** - Test generation
- **zenode:precommit** - Git change validation

### 🚀 Common Use Cases

#### Dedicated Image Analysis with Seer
```bash
# Comprehensive image analysis
zenode:seer "Analyze this image in detail" \
  --images ["/workspace/Desktop/photo.jpg"] \
  --analysis-type detailed

# Professional assessment
zenode:seer "Is this suitable for business use?" \
  --images ["/workspace/headshot.jpg"] \
  --analysis-type professional

# Technical quality check
zenode:seer "Assess photo quality and composition" \
  --images ["/workspace/image.png"] \
  --analysis-type technical
```

#### UI/UX Analysis
```bash
zenode:seer "Review this UI design for usability issues" \
  --images ["/workspace/Desktop/ui-screenshot.png"] \
  --analysis-type detailed \
  --focus-areas "usability,accessibility,visual-hierarchy"
```

#### Code + Visual Documentation
```bash
# Use seer for the visual analysis part
zenode:seer "Analyze this design mockup" \
  --images ["/workspace/project/design.png"] \
  --analysis-type description

# Then analyze code separately
zenode:analyze \
  --files ["/workspace/project/src/component.tsx"] \
  --prompt "Compare code implementation with design described above"
```

#### Debug with Error Screenshots
```bash
zenode:seer "What error is shown in this screenshot?" \
  --images ["/workspace/Downloads/error-screenshot.png"] \
  --analysis-type description \
  --focus-areas "error-messages,UI-state"
```

## Tools and Reasoning Guidance

Remember, you can call zenode tools to help you reason about things more fully, to respond to user queries telling you to think deeply or get multiple perspectives. you can call zenode tools by calling them directly with zenode:thinkdeep, zenode:analyze, zenode:chat, zenode:codereview, zenode:testgen, zenode:grunts, etc. but you can also use the :z command and you should use it when asked for something comprehensive or critical. any context you pass to zenode tools should include the context but also asking them to be critical and skeptical and to have all major assertions discussed and checked.

**IMPORTANT**: Always use `/workspace/` paths when providing files to zenode tools, not local filesystem paths.

## 🤖 Automatic Path Transformation for Claude

### Critical Instruction for Claude Code Assistant

When the user provides file paths or when you need to provide file paths to zenode tools, **ALWAYS automatically transform them** according to these rules:

#### Path Transformation Rules
```bash
# Transform user-provided paths:
/Users/edunc/anything/file.ext  →  /workspace/anything/file.ext
/home/username/anything/file.ext  →  /workspace/anything/file.ext
~/Desktop/image.jpg  →  /workspace/Desktop/image.jpg
./relative/path.txt  →  /workspace/[resolve-to-absolute]/path.txt
```

#### Required Transformation in Every Zenode Call

**Before calling ANY zenode tool with files, transform paths and add workspace tip:**

```bash
# User says: "zenode:chat analyze this /Users/edunc/Desktop/screenshot.png"
# Claude should transform to:
zenode:chat "analyze this" --files ["/workspace/Desktop/screenshot.png"]

# And append this tip to every zenode call:
# 💡 Path automatically transformed from /Users/edunc/Desktop/screenshot.png to container path /workspace/Desktop/screenshot.png
```

#### Template for Every Zenode Call

When calling zenode tools, Claude should ALWAYS:

1. **Transform all paths** from local to container format
2. **Add workspace configuration tip** to the prompt
3. **Explain the transformation** in a brief note

**Example Format:**
```
zenode:chat "Analyze this image for usability issues" --files ["/workspace/Desktop/ui-screenshot.png"]

💡 **Workspace Note:** Path automatically transformed (/Users/edunc/Desktop/ui-screenshot.png → /workspace/Desktop/ui-screenshot.png). Zenode tools run in Docker containers and require /workspace/ paths to access your files.
```

#### Auto-Transform Function for Claude

Use this mental template for every zenode interaction:

```typescript
function transformForZenode(userPath: string): string {
  // Replace home directory path with /workspace
  return userPath.replace(/^\/Users\/[^\/]+/, '/workspace')
                .replace(/^\/home\/[^\/]+/, '/workspace')
                .replace(/^~/, '/workspace');
}

function callZenode(tool: string, prompt: string, files: string[]) {
  const transformedFiles = files.map(transformForZenode);
  return `zenode:${tool} "${prompt}" --files ["${transformedFiles.join('", "')}"]
  
💡 **Workspace Note:** Paths transformed for container access. Files accessible via Docker volume mount.`;
}
```

### Helper Scripts Available

The user can also use these helper scripts:
- `./convert-paths.sh` - Interactive path conversion tool
- `./setup-workspace.sh` - Verify workspace configuration

**REMEMBER: Never use raw local paths with zenode tools. Always transform and explain.**

## 🛠️ Zenode Troubleshooting & Connection Issues

### Status: Zenode Tested & Working ✅

**Last Tested:** 2025-06-18  
**Test Results:** All systems operational - zenode:seer, zenode:chat, and all 13 tools working correctly

### Common Connection Issues & Solutions

#### 1. **Docker Containers Not Running**
**Symptoms:** `zenode:version` fails, no response from zenode tools  
**Root Cause:** Docker containers stopped or not started  
**Solution:**
```bash
cd zenode/
docker-compose ps  # Check container status
docker-compose up -d  # Start containers if stopped
```

#### 2. **API Key Configuration Issues**
**Symptoms:** Tools respond but fail with authentication errors  
**Root Cause:** Missing or invalid API keys in `.env` file  
**Solution:**
```bash
# Check .env file has valid API key
cat .env | grep -E "(OPENROUTER|OPENAI|GEMINI)_API_KEY"

# Verify API key format (should start with sk-or-v1- for OpenRouter)
# Example: OPENROUTER_API_KEY=sk-or-v1-your_key_here
```

#### 3. **Container Health Issues**
**Symptoms:** Containers running but zenode tools timeout or fail  
**Root Cause:** Unhealthy containers, Redis connection issues  
**Solution:**
```bash
# Check container health
docker-compose ps
# Look for "Up (healthy)" status

# Check logs for errors
docker-compose logs zenode-server --tail=50
docker-compose logs redis --tail=20

# Restart unhealthy containers
docker-compose restart zenode-server redis
```

#### 4. **Path Access Issues**
**Symptoms:** zenode:seer or zenode:gopher can't find files  
**Root Cause:** Using local paths instead of container paths  
**Solution:**
```bash
# ❌ WRONG: Local path
zenode:seer "analyze" --images ["/Users/edunc/Desktop/image.jpg"]

# ✅ CORRECT: Container path  
zenode:seer "analyze" --images ["/workspace/Desktop/image.jpg"]

# Verify file access
zenode:gopher --action file_exists --path "/workspace/Desktop/image.jpg"
```

#### 5. **Port Conflicts**
**Symptoms:** Docker containers fail to start, port binding errors  
**Root Cause:** Redis port 6380 already in use  
**Solution:**
```bash
# Check what's using port 6380
lsof -i :6380

# Kill process if needed or change port in docker-compose.yml
docker-compose down
docker-compose up -d
```

#### 6. **MCP Connection Issues**
**Symptoms:** MCP tools not available in Claude Code  
**Root Cause:** MCP server not properly registered  
**Solution:**
```bash
# Verify MCP server is running
zenode:version

# Check MCP server logs
docker-compose logs zenode-server | grep -E "(error|fail)"

# Restart MCP server if needed
docker-compose restart zenode-server
```

### Diagnostic Commands

#### Quick Health Check
```bash
# 1. Verify Docker containers
docker-compose ps

# 2. Test basic zenode functionality  
zenode:version

# 3. Test simple operation
zenode:chat "test - what is 2+2?" --model auto

# 4. Check logs for errors
docker-compose logs zenode-server --tail=20 | grep -E "(error|fail|warn)"
```

#### Comprehensive Diagnostic
```bash
# System status
docker-compose ps
docker-compose logs zenode-server --tail=50
docker-compose logs redis --tail=20

# API configuration
env | grep -E "(OPENROUTER|OPENAI|GEMINI)_API_KEY"

# Network connectivity
docker network ls
docker exec zenode-server ping redis

# File system access
zenode:gopher --action list_directory --path "/workspace"
```

### Prevention Tips

1. **Always check container status** before using zenode tools
2. **Use `/workspace/` paths** for all file operations
3. **Keep API keys secure** and properly formatted in `.env`
4. **Monitor container logs** for early warning signs
5. **Test zenode:version** regularly to verify connectivity

### Emergency Recovery

If zenode becomes completely unresponsive:
```bash
# Nuclear option - full restart
cd zenode/
docker-compose down
docker-compose up -d

# Wait 30 seconds for startup
sleep 30

# Verify recovery
zenode:version
```

**Note:** Based on recent testing, the most common issue is **path confusion** between local filesystem paths and container paths. Always use `/workspace/` prefix for zenode tools.

## 🚀 CLI Mode Implementation - BREAKTHROUGH!

### Major Achievement: Dual MCP/CLI Operation ✅

**Date:** 2025-06-18  
**Status:** Successfully implemented zenode as both MCP server AND command-line tool

#### What Was The Problem?
- Zenode was originally designed as **MCP-only server** that listened on stdin/stdout
- Running `docker exec zenode-server node dist/index.js seer` would start the MCP server and hang waiting for JSON-RPC messages
- No way to run tools directly from command line for testing/debugging

#### The Solution: Intelligent Mode Detection

Added CLI mode detection to `src/index.ts`:

```typescript
async function startZenode() {
  // Check if CLI arguments are provided
  const hasCliArgs = process.argv.length > 2;
  
  if (hasCliArgs) {
    // CLI Mode: node dist/index.js toolname args
    await runCliMode();
  } else {
    // MCP Server Mode: Default behavior for MCP clients
    await main();
  }
}
```

#### CLI Usage Examples

**Basic tool execution:**
```bash
# Run on host machine
node dist/index.js version
node dist/index.js chat '{"prompt": "Hello world"}'

# Run in Docker container  
docker exec zenode-server node dist/index.js version
docker exec zenode-server node dist/index.js seer '{"prompt": "What do you see?", "images": ["demo-output/image.jpg"]}'
```

**Tool execution with JSON output:**
```bash
ZENODE_CLI_OUTPUT=json node dist/index.js version
```

**Debug mode with full logging:**
```bash
ZENODE_CLI_DEBUG=1 node dist/index.js seer '{"prompt": "analyze", "images": ["path/to/image.jpg"]}'
```

#### Benefits of CLI Mode

1. **🔧 Direct Tool Testing**: Test individual tools without MCP protocol overhead
2. **🐛 Debugging**: Easier troubleshooting and development
3. **⚡ Fast Iteration**: Quick testing of model configurations and parameters
4. **📊 Scripting**: Enables automation and batch processing
5. **🔍 Diagnostics**: Direct access to tool execution and error messages

#### Current Status & Findings

**✅ Working Features:**
- CLI mode detection and activation
- Tool registry access (all 13 tools available)
- Version tool execution
- Logging suppression for clean output
- Error handling and exit codes
- Both MCP server and CLI modes functional

**⚠️ Current Issue: Seer Vision Model Selection**
- Seer tool executes but fails with "Unknown model" errors
- Issue appears to be in vision model configuration/selection
- Models like `openai/gpt-4o`, `vision`, `auto` not being recognized
- Custom model configuration may need OpenRouter vision model updates

**📝 Next Steps for Full Seer Functionality:**
1. Update custom_models.json with current OpenRouter vision models (June 2025)
2. Fix model resolution in seer tool for vision capabilities
3. Test with confirmed available models like `gpt-4o`, `claude-3.5-sonnet`
4. Verify OpenRouter API connectivity for vision endpoints

#### Implementation Files Modified

- `src/index.ts`: Added CLI mode detection and execution
- Built with TypeScript compilation and deployed to Docker container
- Maintains backward compatibility with MCP server functionality

This breakthrough enables zenode to work as both a production MCP server AND a development/testing CLI tool!