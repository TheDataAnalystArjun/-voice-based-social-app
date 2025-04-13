# -voice-based-social-app
import { useState, useEffect } from 'react';
import { X, Mic, MicOff, Hand, HandMetal, Crown, MoreHorizontal, Settings, Music } from 'lucide-react';

export default function ClubhouseApp() {
  const [participants, setParticipants] = useState([
    { id: 1, name: "Taylor Swift", avatar: "/api/placeholder/80/80", role: "moderator", speaking: false, muted: false, handRaised: false },
    { id: 2, name: "John Legend", avatar: "/api/placeholder/80/80", role: "speaker", speaking: false, muted: false, handRaised: false },
    { id: 3, name: "Beyoncé", avatar: "/api/placeholder/80/80", role: "speaker", speaking: false, muted: false, handRaised: false },
    { id: 4, name: "Drake", avatar: "/api/placeholder/80/80", role: "listener", speaking: false, muted: true, handRaised: false },
    { id: 5, name: "Adele", avatar: "/api/placeholder/80/80", role: "listener", speaking: false, muted: true, handRaised: false },
    { id: 6, name: "Billie Eilish", avatar: "/api/placeholder/80/80", role: "listener", speaking: false, muted: true, handRaised: false },
    { id: 7, name: "Bruno Mars", avatar: "/api/placeholder/80/80", role: "listener", speaking: false, muted: true, handRaised: false },
    { id: 8, name: "You", avatar: "/api/placeholder/80/80", role: "speaker", speaking: false, muted: false, handRaised: false, isUser: true },
  ]);

  const [modActions, setModActions] = useState({ visible: false, userId: null });

  // Simulation of random speakers talking
  useEffect(() => {
    const interval = setInterval(() => {
      setParticipants(current => {
        const speakerIds = current.filter(p => p.role === "speaker" && !p.muted && !p.isUser).map(p => p.id);
        if (speakerIds.length === 0) return current;
        
        // Randomly select a speaker to talk
        const randomId = speakerIds[Math.floor(Math.random() * speakerIds.length)];
        
        return current.map(p => {
          if (p.id === randomId) {
            return { ...p, speaking: true };
          } else if (p.speaking && !p.isUser) {
            return { ...p, speaking: false };
          }
          return p;
        });
      });
    }, 4000);
    
    return () => clearInterval(interval);
  }, []);

  // Handle mic toggle
  const toggleMic = (userId) => {
    setParticipants(current => 
      current.map(p => 
        p.id === userId ? { ...p, muted: !p.muted, speaking: p.muted ? p.speaking : false } : p
      )
    );
  };

  // Handle hand raise
  const toggleHandRaise = (userId) => {
    setParticipants(current => 
      current.map(p => 
        p.id === userId ? { ...p, handRaised: !p.handRaised } : p
      )
    );
  };

  // Promote listener to speaker
  const promoteToSpeaker = (userId) => {
    setParticipants(current => 
      current.map(p => 
        p.id === userId ? { ...p, role: "speaker", muted: false, handRaised: false } : p
      )
    );
    setModActions({ visible: false, userId: null });
  };

  // Demote speaker to listener
  const demoteToListener = (userId) => {
    setParticipants(current => 
      current.map(p => 
        p.id === userId ? { ...p, role: "listener", muted: true, speaking: false } : p
      )
    );
    setModActions({ visible: false, userId: null });
  };

  // Show/hide moderation actions menu
  const toggleModActions = (userId) => {
    setModActions(current => 
      current.userId === userId && current.visible 
        ? { visible: false, userId: null } 
        : { visible: true, userId }
    );
  };

  return (
    <div className="flex flex-col bg-gray-900 text-white min-h-screen">
      {/* Room Header */}
      <div className="bg-gradient-to-r from-purple-800 to-blue-900 p-4 flex justify-between items-center">
        <div className="flex-1">
          <h1 className="text-xl font-bold">Music Industry Trends in 2025</h1>
          <p className="text-gray-300 text-sm">A discussion with industry pros</p>
        </div>
        <button className="bg-gray-800 hover:bg-gray-700 p-2 rounded-full transition-all">
          <X size={20} />
        </button>
      </div>

      {/* Participants Section */}
      <div className="flex-1 p-4 overflow-y-auto">
        {/* Moderators & Speakers */}
        <div className="mb-8">
          <h2 className="text-gray-400 mb-3 text-sm uppercase tracking-wide">On Stage</h2>
          <div className="grid grid-cols-3 gap-4 md:grid-cols-4">
            {participants
              .filter(p => p.role === "moderator" || p.role === "speaker")
              .map(participant => (
                <div key={participant.id} className="relative">
                  <div 
                    className={`flex flex-col items-center transition-all duration-300 ${
                      participant.speaking ? 'scale-105 opacity-100' : 'opacity-80'
                    }`}
                  >
                    <div className="relative">
                      <div className={`relative transition-all duration-300 ${
                        participant.speaking ? 'ring-4 ring-green-500' : ''
                      } rounded-full mb-2`}>
                        <img 
                          src={participant.avatar} 
                          alt={participant.name} 
                          className="w-16 h-16 rounded-full object-cover"
                        />
                        <div className={`absolute -bottom-1 -right-1 w-6 h-6 rounded-full flex items-center justify-center ${
                          participant.muted ? 'bg-red-500' : participant.speaking ? 'bg-green-500' : 'bg-gray-700'
                        }`}>
                          {participant.muted ? <MicOff size={12} /> : <Mic size={12} />}
                        </div>
                        {participant.role === "moderator" && (
                          <div className="absolute -top-1 -right-1 w-6 h-6 bg-purple-500 rounded-full flex items-center justify-center">
                            <Crown size={12} />
                          </div>
                        )}
                      </div>
                    </div>
                    <span className="text-sm font-medium truncate max-w-full">
                      {participant.name} {participant.isUser ? "(You)" : ""}
                    </span>
                    
                    {participant.isUser ? null : (
                      <button 
                        onClick={() => toggleModActions(participant.id)}
                        className="text-gray-400 hover:text-white mt-1"
                      >
                        <MoreHorizontal size={16} />
                      </button>
                    )}
                    
                    {/* Moderation Actions Menu */}
                    {modActions.visible && modActions.userId === participant.id && (
                      <div className="absolute top-full mt-2 bg-gray-800 rounded-lg shadow-lg p-2 z-10 w-36">
                        {participant.role === "speaker" ? (
                          <button 
                            onClick={() => demoteToListener(participant.id)}
                            className="flex items-center w-full p-2 hover:bg-gray-700 rounded text-left"
                          >
                            <span className="ml-2 text-sm">Move to audience</span>
                          </button>
                        ) : participant.role === "listener" ? (
                          <button 
                            onClick={() => promoteToSpeaker(participant.id)}
                            className="flex items-center w-full p-2 hover:bg-gray-700 rounded text-left"
                          >
                            <span className="ml-2 text-sm">Invite to speak</span>
                          </button>
                        ) : null}
                      </div>
                    )}
                  </div>
                </div>
              ))}
          </div>
        </div>

        {/* Listeners */}
        <div>
          <h2 className="text-gray-400 mb-3 text-sm uppercase tracking-wide">Listeners</h2>
          <div className="grid grid-cols-4 gap-3 md:grid-cols-6">
            {participants
              .filter(p => p.role === "listener")
              .map(participant => (
                <div key={participant.id} className="relative">
                  <div className="flex flex-col items-center">
                    <div className="relative mb-2">
                      <img 
                        src={participant.avatar} 
                        alt={participant.name} 
                        className="w-12 h-12 rounded-full object-cover opacity-80"
                      />
                      {participant.handRaised && (
                        <div className="absolute -top-1 -right-1 w-5 h-5 bg-blue-500 rounded-full flex items-center justify-center">
                          <Hand size={10} />
                        </div>
                      )}
                    </div>
                    <span className="text-xs font-medium truncate max-w-full">{participant.name}</span>
                    
                    {!participant.isUser && (
                      <button 
                        onClick={() => toggleModActions(participant.id)}
                        className="text-gray-400 hover:text-white mt-1"
                      >
                        <MoreHorizontal size={14} />
                      </button>
                    )}
                    
                    {/* Moderation Actions Menu */}
                    {modActions.visible && modActions.userId === participant.id && (
                      <div className="absolute top-full mt-2 bg-gray-800 rounded-lg shadow-lg p-2 z-10 w-36">
                        {participant.role === "listener" ? (
                          <button 
                            onClick={() => promoteToSpeaker(participant.id)}
                            className="flex items-center w-full p-2 hover:bg-gray-700 rounded text-left"
                          >
                            <span className="ml-2 text-sm">Invite to speak</span>
                          </button>
                        ) : null}
                      </div>
                    )}
                  </div>
                </div>
              ))}
          </div>
        </div>
      </div>

      {/* Bottom Control Bar */}
      <div className="bg-gray-800 p-4 flex items-center justify-between">
        <div className="flex gap-3">
          {participants.find(p => p.isUser)?.role === "speaker" ? (
            <button 
              onClick={() => toggleMic(participants.find(p => p.isUser).id)}
              className={`rounded-full p-3 ${
                participants.find(p => p.isUser)?.muted 
                  ? 'bg-red-500 hover:bg-red-600' 
                  : 'bg-green-500 hover:bg-green-600'
              } transition-all`}
            >
              {participants.find(p => p.isUser)?.muted ? <MicOff size={20} /> : <Mic size={20} />}
            </button>
          ) : (
            <button 
              onClick={() => toggleHandRaise(participants.find(p => p.isUser).id)}
              className={`rounded-full p-3 ${
                participants.find(p => p.isUser)?.handRaised 
                  ? 'bg-blue-500 hover:bg-blue-600' 
                  : 'bg-gray-700 hover:bg-gray-600'
              } transition-all`}
            >
              {participants.find(p => p.isUser)?.handRaised ? <HandMetal size={20} /> : <Hand size={20} />}
            </button>
          )}
          
          <button className="p-3 rounded-full bg-gray-700 hover:bg-gray-600">
            <Music size={20} />
          </button>
        </div>
        <button className="bg-gray-700 hover:bg-red-500 text-white px-4 py-2 rounded-full transition-all">
          Leave
        </button>
      </div>
    </div>
  );
}
