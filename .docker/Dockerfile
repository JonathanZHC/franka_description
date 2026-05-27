FROM osrf/ros:jazzy-desktop

ARG DEBIAN_FRONTEND=noninteractive

ARG USER_UID=1000
ARG USER_GID=1000
ARG USERNAME=user

SHELL ["/bin/bash", "-c"]

WORKDIR /workspaces

# ------------------------------------------------------------
# Robust user / group creation
#
# The base image may already contain GID=1000 or UID=1000.
# Therefore, do not blindly run:
#   groupadd --gid 1000 user
# because it may fail with:
#   groupadd: GID '1000' already exists
# ------------------------------------------------------------
RUN set -eux; \
    if getent group "${USER_GID}" >/dev/null; then \
        echo "Group with GID ${USER_GID} already exists: $(getent group ${USER_GID})"; \
    else \
        groupadd --gid "${USER_GID}" "${USERNAME}"; \
    fi; \
    if id -u "${USERNAME}" >/dev/null 2>&1; then \
        echo "User ${USERNAME} already exists."; \
        usermod --gid "${USER_GID}" "${USERNAME}" || true; \
    elif getent passwd "${USER_UID}" >/dev/null; then \
        EXISTING_USER="$(getent passwd ${USER_UID} | cut -d: -f1)"; \
        echo "UID ${USER_UID} already exists as user ${EXISTING_USER}; renaming to ${USERNAME}."; \
        usermod -l "${USERNAME}" "${EXISTING_USER}"; \
        usermod -d "/home/${USERNAME}" -m "${USERNAME}" || true; \
        usermod --gid "${USER_GID}" "${USERNAME}" || true; \
    else \
        useradd --uid "${USER_UID}" --gid "${USER_GID}" -m "${USERNAME}"; \
    fi; \
    mkdir -p -m 0700 "/run/user/${USER_UID}"; \
    mkdir -p -m 0700 "/run/user/${USER_UID}/gdm"; \
    chown -R "${USER_UID}:${USER_GID}" "/run/user/${USER_UID}"; \
    chown -R "${USER_UID}:${USER_GID}" /workspaces

# ------------------------------------------------------------
# Runtime dependencies for franka_description URDF generation
# ------------------------------------------------------------
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ros-jazzy-xacro \
        ros-jazzy-realsense2-description \
        ros-jazzy-joint-state-publisher-gui \
        ros-jazzy-realsense2-camera \
        ros-jazzy-sick-safetyscanners2 \
        ros-jazzy-robot-localization \
    && rm -rf /var/lib/apt/lists/*

ENV XDG_RUNTIME_DIR=/run/user/${USER_UID}

USER ${USERNAME}

RUN echo "source /ros_entrypoint.sh" >> ~/.bashrc
ARG MAX_ROS_DOMAIN_ID=232
RUN echo "export ROS_DOMAIN_ID=100" >> ~/.bashrc
RUN echo "set +e" >> ~/.bashrc