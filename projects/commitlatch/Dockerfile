FROM python:3.12-bookworm

ENV PIP_DISABLE_PIP_VERSION_CHECK=1 \
    LANGSMITH_TRACING=false \
    LANGCHAIN_TRACING_V2=false

# The scanner supplies the target repository as the build context.
COPY . /src
WORKDIR /src

# Fetch build tools and optional example dependencies while networking is available.
RUN python -m pip install --no-cache-dir setuptools wheel \
    && python -m pip install --no-cache-dir . -r examples/requirements-langgraph.txt

RUN python -m unittest discover -s tests -v \
    && python examples/quickstart.py \
    && python examples/langgraph_recovery.py \
    && python examples/langgraph_recovery.py --opaque

CMD ["bash"]
